---
layout: post
title:  POSIX多线程程序设计 几种线程模型范例
category: bash
---

最近看了下《POSIX多线程程序设计》，其中第4章"使用线程的几种方式"提到的几种线程模型很经典，这里将其范例摘抄至此。

##流水线 
	/*
	 * pipe.c
	 *
	 * Simple demonstration of a pipeline. main() is a loop that
	 * feeds the pipeline with integer values. Each stage of the
	 * pipeline increases the integer by one before passing it along
	 * to the next. Entering the command "=" reads the pipeline
	 * result. (Notice that too many '=' commands will hang.)
	 */
	#include <pthread.h>
	#include "errors.h"

	/*
	 * Internal structure describing a "stage" in the
	 * pipeline. One for each thread, plus a "result
	 * stage" where the final thread can stash the value.
	 */
	typedef struct stage_tag {
		pthread_mutex_t     mutex;          /* Protect data */
		pthread_cond_t      avail;          /* Data available */
		pthread_cond_t      ready;          /* Ready for data */
		int                 data_ready;     /* Data present */
		long                data;           /* Data to process */
		pthread_t           thread;         /* Thread for stage */
		struct stage_tag    *next;          /* Next stage */
	} stage_t;

	/*
	 * External structure representing the entire
	 * pipeline.
	 */
	typedef struct pipe_tag {
		pthread_mutex_t     mutex;          /* Mutex to protect pipe */
		stage_t             *head;          /* First stage */
		stage_t             *tail;          /* Final stage */
		int                 stages;         /* Number of stages */
		int                 active;         /* Active data elements */
	} pipe_t;

	/*
	 * Internal function to send a "message" to the
	 * specified pipe stage. Threads use this to pass
	 * along the modified data item.
	 */
	int pipe_send (stage_t *stage, long data)
	{
		int status;

		status = pthread_mutex_lock (&stage->mutex);
		if (status != 0)
			return status;
		/*
		 * If there's data in the pipe stage, wait for it
		 * to be consumed.
		 */
		while (stage->data_ready) {
			status = pthread_cond_wait (&stage->ready, &stage->mutex);
			if (status != 0) {
				pthread_mutex_unlock (&stage->mutex);
				return status;
			}
		}

		/*
		 * Send the new data
		 */
		stage->data = data;
		stage->data_ready = 1;
		status = pthread_cond_signal (&stage->avail);
		if (status != 0) {
			pthread_mutex_unlock (&stage->mutex);
			return status;
		}
		status = pthread_mutex_unlock (&stage->mutex);
		return status;
	}

	/*
	 * The thread start routine for pipe stage threads.
	 * Each will wait for a data item passed from the
	 * caller or the previous stage, modify the data
	 * and pass it along to the next (or final) stage.
	 */
	void *pipe_stage (void *arg)
	{
		stage_t *stage = (stage_t*)arg;
		stage_t *next_stage = stage->next;
		int status;

		status = pthread_mutex_lock (&stage->mutex);
		if (status != 0)
			err_abort (status, "Lock pipe stage");
		while (1) {
			while (stage->data_ready != 1) {
				status = pthread_cond_wait (&stage->avail, &stage->mutex);
				if (status != 0)
					err_abort (status, "Wait for previous stage");
			}
			pipe_send (next_stage, stage->data + 1);
			stage->data_ready = 0;
			status = pthread_cond_signal (&stage->ready);
			if (status != 0)
				err_abort (status, "Wake next stage");
		}
		/*
		 * Notice that the routine never unlocks the stage->mutex.
		 * The call to pthread_cond_wait implicitly unlocks the
		 * mutex while the thread is waiting, allowing other threads
		 * to make progress. Because the loop never terminates, this
		 * function has no need to unlock the mutex explicitly.
		 */
	}

	/*
	 * External interface to create a pipeline. All the
	 * data is initialized and the threads created. They'll
	 * wait for data.
	 */
	int pipe_create (pipe_t *pipe, int stages)
	{
		int pipe_index;
		stage_t **link = &pipe->head, *new_stage, *stage;
		int status;

		status = pthread_mutex_init (&pipe->mutex, NULL);
		if (status != 0)
			err_abort (status, "Init pipe mutex");
		pipe->stages = stages;
		pipe->active = 0;

		for (pipe_index = 0; pipe_index <= stages; pipe_index++) {
			new_stage = (stage_t*)malloc (sizeof (stage_t));
			if (new_stage == NULL)
				errno_abort ("Allocate stage");
			status = pthread_mutex_init (&new_stage->mutex, NULL);
			if (status != 0)
				err_abort (status, "Init stage mutex");
			status = pthread_cond_init (&new_stage->avail, NULL);
			if (status != 0)
				err_abort (status, "Init avail condition");
			status = pthread_cond_init (&new_stage->ready, NULL);
			if (status != 0)
				err_abort (status, "Init ready condition");
			new_stage->data_ready = 0;
			*link = new_stage;
			link = &new_stage->next;
		}

		*link = (stage_t*)NULL;     /* Terminate list */
		pipe->tail = new_stage;     /* Record the tail */

		/*
		 * Create the threads for the pipe stages only after all
		 * the data is initialized (including all links). Note
		 * that the last stage doesn't get a thread, it's just
		 * a receptacle for the final pipeline value.
		 *
		 * At this point, proper cleanup on an error would take up
		 * more space than worthwhile in a "simple example", so
		 * instead of cancelling and detaching all the threads
		 * already created, plus the synchronization object and
		 * memory cleanup done for earlier errors, it will simply
		 * abort.
		 */
		for (   stage = pipe->head;
				stage->next != NULL;
				stage = stage->next) {
			status = pthread_create (
				&stage->thread, NULL, pipe_stage, (void*)stage);
			if (status != 0)
				err_abort (status, "Create pipe stage");
		}
		return 0;
	}

	/*
	 * External interface to start a pipeline by passing
	 * data to the first stage. The routine returns while
	 * the pipeline processes in parallel. Call the
	 * pipe_result return to collect the final stage values
	 * (note that the pipe will stall when each stage fills,
	 * until the result is collected).
	 */
	int pipe_start (pipe_t *pipe, long value)
	{
		int status;

		status = pthread_mutex_lock (&pipe->mutex);
		if (status != 0)
			err_abort (status, "Lock pipe mutex");
		pipe->active++;
		status = pthread_mutex_unlock (&pipe->mutex);
		if (status != 0)
			err_abort (status, "Unlock pipe mutex");
		pipe_send (pipe->head, value);
		return 0;
	}

	/*
	 * Collect the result of the pipeline. Wait for a
	 * result if the pipeline hasn't produced one.
	 */
	int pipe_result (pipe_t *pipe, long *result)
	{
		stage_t *tail = pipe->tail;
		long value;
		int empty = 0;
		int status;

		status = pthread_mutex_lock (&pipe->mutex);
		if (status != 0)
			err_abort (status, "Lock pipe mutex");
		if (pipe->active <= 0)
			empty = 1;
		else
			pipe->active--;

		status = pthread_mutex_unlock (&pipe->mutex);
		if (status != 0)
			err_abort (status, "Unlock pipe mutex");
		if (empty)
			return 0;

		pthread_mutex_lock (&tail->mutex);
		while (!tail->data_ready)
			pthread_cond_wait (&tail->avail, &tail->mutex);
		*result = tail->data;
		tail->data_ready = 0;
		pthread_cond_signal (&tail->ready);
		pthread_mutex_unlock (&tail->mutex);    
		return 1;
	}

	/*
	 * The main program to "drive" the pipeline...
	 */
	int main (int argc, char *argv[])
	{
		pipe_t my_pipe;
		long value, result;
		int status;
		char line[128];

		pipe_create (&my_pipe, 10);
		printf ("Enter integer values, or \"=\" for next result\n");

		while (1) {
			printf ("Data> ");
			if (fgets (line, sizeof (line), stdin) == NULL) exit (0);
			if (strlen (line) <= 1) continue;
			if (strlen (line) <= 2 && line[0] == '=') {
				if (pipe_result (&my_pipe, &result))
					printf ("Result is %ld\n", result);
				else
					printf ("Pipe is empty\n");
			} else {
				if (sscanf (line, "%ld", &value) < 1)
					fprintf (stderr, "Enter an integer value\n");
				else
					pipe_start (&my_pipe, value);
			}
		}
	}

##工作组 
	/*
	 * crew.c
	 *
	 * Demonstrate a work crew implementing a simple parallel search
	 * through a directory tree.
	 *
	 * Special notes: On a Solaris 2.5 uniprocessor, this test will
	 * not produce interleaved output unless extra LWPs are created
	 * by calling thr_setconcurrency(), because threads are not
	 * timesliced.
	 */
	#include <sys/types.h>
	#include <pthread.h>
	#include <sys/stat.h>
	#include <dirent.h>
	#include "errors.h"

	#define CREW_SIZE       4

	/*
	 * Queued items of work for the crew. One is queued by
	 * crew_start, and each worker may queue additional items.
	 */
	typedef struct work_tag {
		struct work_tag     *next;          /* Next work item */
		char                *path; /* Directory or file */
		char                *string;        /* Search string */
	} work_t, *work_p;

	/*
	 * One of these is initialized for each worker thread in the
	 * crew. It contains the "identity" of each worker.
	 */
	typedef struct worker_tag {
		int                 index;          /* Thread's index */
		pthread_t           thread;         /* Thread for stage */
		struct crew_tag     *crew;          /* Pointer to crew */
	} worker_t, *worker_p;

	/*
	 * The external "handle" for a work crew. Contains the
	 * crew synchronization state and staging area.
	 */
	typedef struct crew_tag {
		int                 crew_size;      /* Size of array */
		worker_t            crew[CREW_SIZE];/* Crew members */
		long                work_count;     /* Count of work items */
		work_t              *first, *last;  /* First & last work item */
		pthread_mutex_t     mutex;          /* Mutex for crew data */
		pthread_cond_t      done;           /* Wait for crew done */
		pthread_cond_t      go;             /* Wait for work */
	} crew_t, *crew_p;

	size_t  path_max;                       /* Filepath length */
	size_t  name_max;                       /* Name length */

	/*
	 * The thread start routine for crew threads. Waits until "go"
	 * command, processes work items until requested to shut down.
	 */
	void *worker_routine (void *arg)
	{
		worker_p mine = (worker_t*)arg;
		crew_p crew = mine->crew;
		work_p work, new_work;
		struct stat filestat;
		struct dirent *entry;
		int status;

		/*
		 * "struct dirent" is funny, because POSIX doesn't require
		 * the definition to be more than a header for a variable
		 * buffer. Thus, allocate a "big chunk" of memory, and use
		 * it as a buffer.
		 */
		entry = (struct dirent*)malloc (
			sizeof (struct dirent) + name_max);
		if (entry == NULL)
			errno_abort ("Allocating dirent");
		
		status = pthread_mutex_lock (&crew->mutex);
		if (status != 0)
			err_abort (status, "Lock crew mutex");

		/*
		 * There won't be any work when the crew is created, so wait
		 * until something's put on the queue.
		 */
		while (crew->work_count == 0) {
			status = pthread_cond_wait (&crew->go, &crew->mutex);
			if (status != 0)
				err_abort (status, "Wait for go");
		}

		status = pthread_mutex_unlock (&crew->mutex);
		if (status != 0)
			err_abort (status, "Unlock mutex");

		DPRINTF (("Crew %d starting\n", mine->index));

		/*
		 * Now, as long as there's work, keep doing it.
		 */
		while (1) {
			/*
			 * Wait while there is nothing to do, and
			 * the hope of something coming along later. If
			 * crew->first is NULL, there's no work. But if
			 * crew->work_count goes to zero, we're done.
			 */
			status = pthread_mutex_lock (&crew->mutex);
			if (status != 0)
				err_abort (status, "Lock crew mutex");

			DPRINTF (("Crew %d top: first is %#lx, count is %d\n",
					  mine->index, crew->first, crew->work_count));
			while (crew->first == NULL) {
				status = pthread_cond_wait (&crew->go, &crew->mutex);
				if (status != 0)
					err_abort (status, "Wait for work");
			}

			DPRINTF (("Crew %d woke: %#lx, %d\n",
					  mine->index, crew->first, crew->work_count));

			/*
			 * Remove and process a work item
			 */
			work = crew->first;
			crew->first = work->next;
			if (crew->first == NULL)
				crew->last = NULL;

			DPRINTF (("Crew %d took %#lx, leaves first %#lx, last %#lx\n",
					  mine->index, work, crew->first, crew->last));

			status = pthread_mutex_unlock (&crew->mutex);
			if (status != 0)
				err_abort (status, "Unlock mutex");

			/*
			 * We have a work item. Process it, which may involve
			 * queuing new work items.
			 */
			status = lstat (work->path, &filestat);

			if (S_ISLNK (filestat.st_mode))
				printf (
					"Thread %d: %s is a link, skipping.\n",
					mine->index,
					work->path);
			else if (S_ISDIR (filestat.st_mode)) {
				DIR *directory;
				struct dirent *result;

				/*
				 * If the file is a directory, search it and place
				 * all files onto the queue as new work items.
				 */
				directory = opendir (work->path);
				if (directory == NULL) {
					fprintf (
						stderr, "Unable to open directory %s: %d (%s)\n",
						work->path,
						errno, strerror (errno));
					continue;
				}
				
				while (1) {
					status = readdir_r (directory, entry, &result);
					if (status != 0) {
						fprintf (
							stderr,
							"Unable to read directory %s: %d (%s)\n",
							work->path,
							status, strerror (status));
						break;
					}
					if (result == NULL)
						break;              /* End of directory */
					
					/*
					 * Ignore "." and ".." entries.
					 */
					if (strcmp (entry->d_name, ".") == 0)
						continue;
					if (strcmp (entry->d_name, "..") == 0)
						continue;
					new_work = (work_p)malloc (sizeof (work_t));
					if (new_work == NULL)
						errno_abort ("Unable to allocate space");
					new_work->path = (char*)malloc (path_max);
					if (new_work->path == NULL)
						errno_abort ("Unable to allocate path");
					strcpy (new_work->path, work->path);
					strcat (new_work->path, "/");
					strcat (new_work->path, entry->d_name);
					new_work->string = work->string;
					new_work->next = NULL;
					status = pthread_mutex_lock (&crew->mutex);
					if (status != 0)
						err_abort (status, "Lock mutex");
					if (crew->first == NULL) {
						crew->first = new_work;
						crew->last = new_work;
					} else {
						crew->last->next = new_work;
						crew->last = new_work;
					}
					crew->work_count++;
					DPRINTF ((
						"Crew %d: add work %#lx, first %#lx, last %#lx, %d\n",
						mine->index, new_work, crew->first,
						crew->last, crew->work_count));
					status = pthread_cond_signal (&crew->go);
					status = pthread_mutex_unlock (&crew->mutex);
					if (status != 0)
						err_abort (status, "Unlock mutex");
				}
				
				closedir (directory);
			} else if (S_ISREG (filestat.st_mode)) {
				FILE *search;
				char buffer[256], *bufptr, *search_ptr;

				/*
				 * If this is a file, not a directory, then search
				 * it for the string.
				 */
				search = fopen (work->path, "r");
				if (search == NULL)
					fprintf (
						stderr, "Unable to open %s: %d (%s)\n",
						work->path,
						errno, strerror (errno));
				else {

					while (1) {
						bufptr = fgets (
							buffer, sizeof (buffer), search);
						if (bufptr == NULL) {
							if (feof (search))
								break;
							if (ferror (search)) {
								fprintf (
									stderr,
									"Unable to read %s: %d (%s)\n",
									work->path,
									errno, strerror (errno));
								break;
							}
						}
						search_ptr = strstr (buffer, work->string);
						if (search_ptr != NULL) {
							flockfile (stdout);
							printf (
								"Thread %d found \"%s\" in %s\n",
								mine->index, work->string, work->path);
	#if 0
							printf ("%s\n", buffer);
	#endif
							funlockfile (stdout);
							break;
						}
					}
					fclose (search);
				}
			} else
				fprintf (
					stderr,
					"Thread %d: %s is type %o (%s))\n",
					mine->index,
					work->path,
					filestat.st_mode & S_IFMT,
					(S_ISFIFO (filestat.st_mode) ? "FIFO"
					 : (S_ISCHR (filestat.st_mode) ? "CHR"
						: (S_ISBLK (filestat.st_mode) ? "BLK"
						   : (S_ISSOCK (filestat.st_mode) ? "SOCK"
							  : "unknown")))));

			free (work->path);              /* Free path buffer */
			free (work);                    /* We're done with this */

			/*
			 * Decrement count of outstanding work items, and wake
			 * waiters (trying to collect results or start a new
			 * calculation) if the crew is now idle.
			 *
			 * It's important that the count be decremented AFTER
			 * processing the current work item. That ensures the
			 * count won't go to 0 until we're really done.
			 */
			status = pthread_mutex_lock (&crew->mutex);
			if (status != 0)
				err_abort (status, "Lock crew mutex");

			crew->work_count--;
			DPRINTF (("Crew %d decremented work to %d\n", mine->index,
					  crew->work_count));
			if (crew->work_count <= 0) {
				DPRINTF (("Crew thread %d done\n", mine->index));
				status = pthread_cond_broadcast (&crew->done);
				if (status != 0)
					err_abort (status, "Wake waiters");
				status = pthread_mutex_unlock (&crew->mutex);
				if (status != 0)
					err_abort (status, "Unlock mutex");
				break;
			}

			status = pthread_mutex_unlock (&crew->mutex);
			if (status != 0)
				err_abort (status, "Unlock mutex");

		}

		free (entry);
		return NULL;
	}

	/*
	 * Create a work crew.
	 */
	int crew_create (crew_t *crew, int crew_size)
	{
		int crew_index;
		int status;

		/*
		 * We won't create more than CREW_SIZE members
		 */
		if (crew_size > CREW_SIZE)
			return EINVAL;

		crew->crew_size = crew_size;
		crew->work_count = 0;
		crew->first = NULL;
		crew->last = NULL;

		/*
		 * Initialize synchronization objects
		 */
		status = pthread_mutex_init (&crew->mutex, NULL);
		if (status != 0)
			return status;
		status = pthread_cond_init (&crew->done, NULL);
		if (status != 0)
			return status;
		status = pthread_cond_init (&crew->go, NULL);
		if (status != 0)
			return status;

		/*
		 * Create the worker threads.
		 */
		for (crew_index = 0; crew_index < CREW_SIZE; crew_index++) {
			crew->crew[crew_index].index = crew_index;
			crew->crew[crew_index].crew = crew;
			status = pthread_create (&crew->crew[crew_index].thread,
				NULL, worker_routine, (void*)&crew->crew[crew_index]);
			if (status != 0)
				err_abort (status, "Create worker");
		}
		return 0;
	}

	/*
	 * Pass a file path to a work crew previously created
	 * using crew_create
	 */
	int crew_start (
		crew_p crew,
		char *filepath,
		char *search)
	{
		work_p request;
		int status;

		status = pthread_mutex_lock (&crew->mutex);
		if (status != 0)
			return status;

		/*
		 * If the crew is busy, wait for them to finish.
		 */
		while (crew->work_count > 0) {
			status = pthread_cond_wait (&crew->done, &crew->mutex);
			if (status != 0) {
				pthread_mutex_unlock (&crew->mutex);
				return status;
			}
		}

		errno = 0;
		path_max = pathconf (filepath, _PC_PATH_MAX);
		if (path_max == -1) {
			if (errno == 0)
				path_max = 1024;             /* "No limit" */
			else
				errno_abort ("Unable to get PATH_MAX");
		}
		errno = 0;
		name_max = pathconf (filepath, _PC_NAME_MAX);
		if (name_max == -1) {
			if (errno == 0)
				name_max = 256;             /* "No limit" */
			else
				errno_abort ("Unable to get NAME_MAX");
		}
		DPRINTF ((
			"PATH_MAX for %s is %ld, NAME_MAX is %ld\n",
			filepath, path_max, name_max));
		path_max++;                         /* Add null byte */
		name_max++;                         /* Add null byte */
		request = (work_p)malloc (sizeof (work_t));
		if (request == NULL)
			errno_abort ("Unable to allocate request");
		DPRINTF (("Requesting %s\n", filepath));
		request->path = (char*)malloc (path_max);
		if (request->path == NULL)
			errno_abort ("Unable to allocate path");
		strcpy (request->path, filepath);
		request->string = search;
		request->next = NULL;
		if (crew->first == NULL) {
			crew->first = request;
			crew->last = request;
		} else {
			crew->last->next = request;
			crew->last = request;
		}

		crew->work_count++;
		status = pthread_cond_signal (&crew->go);
		if (status != 0) {
			free (crew->first);
			crew->first = NULL;
			crew->work_count = 0;
			pthread_mutex_unlock (&crew->mutex);
			return status;
		}
		while (crew->work_count > 0) {
			status = pthread_cond_wait (&crew->done, &crew->mutex);
			if (status != 0)
				err_abort (status, "waiting for crew to finish");
		}
		status = pthread_mutex_unlock (&crew->mutex);
		if (status != 0)
			err_abort (status, "Unlock crew mutex");
		return 0;
	}

	/*
	 * The main program to "drive" the crew...
	 */
	int main (int argc, char *argv[])
	{
		crew_t my_crew;
		char line[128], *next;
		int status;

		if (argc < 3) {
			fprintf (stderr, "Usage: %s string path\n", argv[0]);
			return -1;
		}

	#ifdef sun
		/*
		 * On Solaris 2.5, threads are not timesliced. To ensure
		 * that our threads can run concurrently, we need to
		 * increase the concurrency level to CREW_SIZE.
		 */
		DPRINTF (("Setting concurrency level to %d\n", CREW_SIZE));
		thr_setconcurrency (CREW_SIZE);
	#endif
		status = crew_create (&my_crew, CREW_SIZE);
		if (status != 0)
			err_abort (status, "Create crew");

		status = crew_start (&my_crew, argv[2], argv[1]);
		if (status != 0)
			err_abort (status, "Start crew");

		return 0;
	}


##客户/服务器
	/*
	 * server.c
	 *
	 * Demonstrate a client/server threading model.
	 *
	 * Special notes: On a Solaris system, call thr_setconcurrency()
	 * to allow interleaved thread execution, since threads are not
	 * timesliced.
	 */
	#include <pthread.h>
	#include <math.h>
	#include "errors.h"

	#define CLIENT_THREADS  4               /* Number of clients */

	#define REQ_READ        1               /* Read with prompt */
	#define REQ_WRITE       2               /* Write */
	#define REQ_QUIT        3               /* Quit server */

	/*
	 * Internal to server "package" -- one for each request.
	 */
	typedef struct request_tag {
		struct request_tag  *next;          /* Link to next */
		int                 operation;      /* Function code */
		int                 synchronous;    /* Non-zero if synchronous */
		int                 done_flag;      /* Predicate for wait */
		pthread_cond_t      done;           /* Wait for completion */
		char                prompt[32];     /* Prompt string for reads */
		char                text[128];      /* Read/write text */
	} request_t;

	/*
	 * Static context for the server
	 */
	typedef struct tty_server_tag {
		request_t           *first;
		request_t           *last;
		int                 running;
		pthread_mutex_t     mutex;
		pthread_cond_t      request;
	} tty_server_t;

	tty_server_t tty_server = {
		NULL, NULL, 0,
		PTHREAD_MUTEX_INITIALIZER, PTHREAD_COND_INITIALIZER};

	/*
	 * Main program data
	 */

	int client_threads;
	pthread_mutex_t client_mutex = PTHREAD_MUTEX_INITIALIZER;
	pthread_cond_t clients_done = PTHREAD_COND_INITIALIZER;

	/*
	 * The server start routine. It waits for a request to appear
	 * in tty_server.requests using the request condition variable.
	 * It processes requests in FIFO order. If a request is marked
	 * "synchronous" (synchronous != 0), the server will set done_flag
	 * and signal the request's condition variable. The client is
	 * responsible for freeing the request. If the request was not
	 * synchronous, the server will free the request on completion.
	 */
	void *tty_server_routine (void *arg)
	{
		static pthread_mutex_t prompt_mutex = PTHREAD_MUTEX_INITIALIZER;
		request_t *request;
		int operation, len;
		int status;

		while (1) {
			status = pthread_mutex_lock (&tty_server.mutex);
			if (status != 0)
				err_abort (status, "Lock server mutex");

			/*
			 * Wait for data
			 */
			while (tty_server.first == NULL) {
				status = pthread_cond_wait (
					&tty_server.request, &tty_server.mutex);
				if (status != 0)
					err_abort (status, "Wait for request");
			}
			request = tty_server.first;
			tty_server.first = request->next;
			if (tty_server.first == NULL)
				tty_server.last = NULL;
			status = pthread_mutex_unlock (&tty_server.mutex);
			if (status != 0)
				err_abort (status, "Unlock server mutex");

			/*
			 * Process the data
			 */
			operation = request->operation;
			switch (operation) {
				case REQ_QUIT:
					break;
				case REQ_READ:
					if (strlen (request->prompt) > 0)
						printf (request->prompt);
					if (fgets (request->text, 128, stdin) == NULL)
						request->text[0] = '\0';
					/*
					 * Because fgets returns the newline, and we don't want it,
					 * we look for it, and turn it into a null (truncating the
					 * input) if found. It should be the last character, if it is
					 * there.
					 */
					len = strlen (request->text);
					if (len > 0 && request->text[len-1] == '\n')
						request->text[len-1] = '\0';
					break;
				case REQ_WRITE:
					puts (request->text);
					break;
				default:
					break;
			}
			if (request->synchronous) {
				status = pthread_mutex_lock (&tty_server.mutex);
				if (status != 0)
					err_abort (status, "Lock server mutex");
				request->done_flag = 1;
				status = pthread_cond_signal (&request->done);
				if (status != 0)
					err_abort (status, "Signal server condition");
				status = pthread_mutex_unlock (&tty_server.mutex);
				if (status != 0)
					err_abort (status, "Unlock server mutex");
			} else
				free (request);
			if (operation == REQ_QUIT)
				break;
		}
		return NULL;
	}

	/*
	 * Request an operation
	 */
	void tty_server_request (
		int         operation,
		int         sync,
		const char  *prompt,
		char        *string)
	{
		request_t *request;
		int status;

		status = pthread_mutex_lock (&tty_server.mutex);
		if (status != 0)
			err_abort (status, "Lock server mutex");
		if (!tty_server.running) {
			pthread_t thread;
			pthread_attr_t detached_attr;

			status = pthread_attr_init (&detached_attr);
			if (status != 0)
				err_abort (status, "Init attributes object");
			status = pthread_attr_setdetachstate (
				&detached_attr, PTHREAD_CREATE_DETACHED);
			if (status != 0)
				err_abort (status, "Set detach state");
			tty_server.running = 1;
			status = pthread_create (&thread, &detached_attr,
				tty_server_routine, NULL);
			if (status != 0)
				err_abort (status, "Create server");

			/*
			 * Ignore an error in destroying the attributes object.
			 * It's unlikely to fail, there's nothing useful we can
			 * do about it, and it's not worth aborting the program
			 * over it.
			 */
			pthread_attr_destroy (&detached_attr);
		}

		/*
		 * Create and initialize a request structure.
		 */
		request = (request_t*)malloc (sizeof (request_t));
		if (request == NULL)
			errno_abort ("Allocate request");
		request->next = NULL;
		request->operation = operation;
		request->synchronous = sync;
		if (sync) {
			request->done_flag = 0;
			status = pthread_cond_init (&request->done, NULL);
			if (status != 0)
				err_abort (status, "Init request condition");
		}
		if (prompt != NULL)
			strncpy (request->prompt, prompt, 32);
		else
			request->prompt[0] = '\0';
		if (operation == REQ_WRITE && string != NULL)
			strncpy (request->text, string, 128);
		else
			request->text[0] = '\0';

		/*
		 * Add the request to the queue, maintaining the first and
		 * last pointers.
		 */
		if (tty_server.first == NULL) {
			tty_server.first = request;
			tty_server.last = request;
		} else {
			(tty_server.last)->next = request;
			tty_server.last = request;
		}

		/*
		 * Tell the server that a request is available.
		 */
		status = pthread_cond_signal (&tty_server.request);
		if (status != 0)
			err_abort (status, "Wake server");

		/*
		 * If the request was "synchronous", then wait for a reply.
		 */
		if (sync) {
			while (!request->done_flag) {
				status = pthread_cond_wait (
					&request->done, &tty_server.mutex);
				if (status != 0)
					err_abort (status, "Wait for sync request");
			}
			if (operation == REQ_READ) {
				if (strlen (request->text) > 0)
					strcpy (string, request->text);
				else
					string[0] = '\0';
			}
			status = pthread_cond_destroy (&request->done);
			if (status != 0)
				err_abort (status, "Destroy request condition");
			free (request);
		}
		status = pthread_mutex_unlock (&tty_server.mutex);
		if (status != 0)
			err_abort (status, "Unlock mutex");
	}

	/*
	 * Client routine -- multiple copies will request server.
	 */
	void *client_routine (void *arg)
	{
		int my_number = (int)arg, loops;
		char prompt[32];
		char string[128], formatted[128];
		int status;

		sprintf (prompt, "Client %d> ", my_number);
		while (1) {
			tty_server_request (REQ_READ, 1, prompt, string);
			if (strlen (string) == 0)
				break;
			for (loops = 0; loops < 4; loops++) {
				sprintf (
					formatted, "(%d#%d) %s", my_number, loops, string);
				tty_server_request (REQ_WRITE, 0, NULL, formatted);
				sleep (1);
			}
		}
		status = pthread_mutex_lock (&client_mutex);
		if (status != 0)
			err_abort (status, "Lock client mutex");
		client_threads--;
		if (client_threads <= 0) {
			status = pthread_cond_signal (&clients_done);
			if (status != 0)
				err_abort (status, "Signal clients done");
		}
		status = pthread_mutex_unlock (&client_mutex);
		if (status != 0)
			err_abort (status, "Unlock client mutex");
		return NULL;
	}

	int main (int argc, char *argv[])
	{
		pthread_t thread;
		int count;
		int status;

	#ifdef sun
		/*
		 * On Solaris 2.5, threads are not timesliced. To ensure
		 * that our threads can run concurrently, we need to
		 * increase the concurrency level to CLIENT_THREADS.
		 */
		DPRINTF (("Setting concurrency level to %d\n", CLIENT_THREADS));
		thr_setconcurrency (CLIENT_THREADS);
	#endif

		/*
		 * Create CLIENT_THREADS clients.
		 */
		client_threads = CLIENT_THREADS;
		for (count = 0; count < client_threads; count++) {
			status = pthread_create (&thread, NULL,
				client_routine, (void*)count);
			if (status != 0)
				err_abort (status, "Create client thread");
		}
		status = pthread_mutex_lock (&client_mutex);
		if (status != 0)
			err_abort (status, "Lock client mutex");
		while (client_threads > 0) {
			status = pthread_cond_wait (&clients_done, &client_mutex);
			if (status != 0)
				err_abort (status, "Wait for clients to finish");
		}
		status = pthread_mutex_unlock (&client_mutex);
		if (status != 0)
			err_abort (status, "Unlock client mutex");
		printf ("All clients done\n");
		tty_server_request (REQ_QUIT, 1, NULL, NULL);
		return 0;
	}

##参考
1. <http://product.china-pub.com/12167#ml>
1. <http://ptgmedia.pearsoncmg.com/images/0201633922/sourcecode/pipe.c>
1. <http://ptgmedia.pearsoncmg.com/images/0201633922/sourcecode/crew.c>
1. <http://ptgmedia.pearsoncmg.com/images/0201633922/sourcecode/server.c>
