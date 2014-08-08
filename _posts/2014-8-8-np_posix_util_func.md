---
layout: post
title: 一些不具移植性的有用函数
category: linux
---

##[accept4](http://man7.org/linux/man-pages/man2/accept.2.html)
在accept时，直接设置SOCK_NONBLOCK/SOCK_CLOEXEC

	int accept4(int sockfd, struct sockaddr *addr,
		           socklen_t *addrlen, int flags);
		           
		           
##[socket](http://man7.org/linux/man-pages/man2/socket.2.html)
int socket(int domain, int type, int protocol);

	Since Linux 2.6.27, the type argument serves a second purpose: in
	       addition to specifying a socket type, it may include the bitwise OR
	       of any of the following values, to modify the behavior of socket():

	       SOCK_NONBLOCK   Set the O_NONBLOCK file status flag on the new open
		               file description.  Using this flag saves extra calls
		               to fcntl(2) to achieve the same result.

	       SOCK_CLOEXEC    Set the close-on-exec (FD_CLOEXEC) flag on the new
		               file descriptor.  See the description of the
		               O_CLOEXEC flag in open(2) for reasons why this may be
		               useful.
		               
		               
##[open](http://man7.org/linux/man-pages/man2/open.2.html)
int open(const char *pathname, int flags);  

int open(const char *pathname, int flags, mode_t mode);

	O_CLOEXEC (since Linux 2.6.23)
		      Enable the close-on-exec flag for the new file descriptor.
		      Specifying this flag permits a program to avoid additional
		      fcntl(2) F_SETFD operations to set the FD_CLOEXEC flag.
		      
		      
##[pipe2](http://man7.org/linux/man-pages/man2/pipe.2.html)
int pipe2(int pipefd[2], int flags);

	O_CLOEXEC
		      Set the close-on-exec (FD_CLOEXEC) flag on the two new file
		      descriptors.  See the description of the same flag in open(2)
		      for reasons why this may be useful.
	O_NONBLOCK
		      Set the O_NONBLOCK file status flag on the two new open file
		      descriptions.  Using this flag saves extra calls to fcntl(2)
		      to achieve the same result.

##[socketpair](http://man7.org/linux/man-pages/man2/socketpair.2.html)
int socketpair(int domain, int type, int protocol, int sv[2]);

	Since Linux 2.6.27, socketpair() supports the SOCK_NONBLOCK and
	       SOCK_CLOEXEC flags described in socket(2).


##[pthread_tryjoin_np](http://man7.org/linux/man-pages/man2/socketpair.2.html)
int pthread_tryjoin_np(pthread_t thread, void **retval);

int pthread_timedjoin_np(pthread_t thread, void **retval,
                        const struct timespec *abstime);
                        
	The pthread_tryjoin_np() function performs a nonblocking join with
	the thread thread, returning the exit status of the thread in
	*retval.  If thread has not yet terminated, then instead of blocking,
	as is done by pthread_join(3), the call returns an error.

	The pthread_timedjoin_np() function performs a join-with-timeout.  If
	thread has not yet terminated, then the call blocks until a maximum
	time, specified in abstime.  If the timeout expires before thread
	terminates, the call returns an error.  The abstime argument is a
	structure of the following form, specifying an absolute time measured
	since the Epoch (see time(2)):

	   struct timespec {
	       time_t tv_sec;     /* seconds */
	       long   tv_nsec;    /* nanoseconds */
	   };
