---
layout: post
title: Linux下的定时器
category: linux
---

        //============================================================================
        // Name        : test.cpp
        // Author      : king(邱金武)
        // Version     :
        // Copyright   : Your copyright notice
        // Description : Hello World in C++, Ansi-style
        //============================================================================


        #include <iostream>
        #include <stdio.h>
        #include <unistd.h>

        #include <signal.h>
        #include <time.h>

        class Timer{
        public:
            typedef void (*CB)(void*arg);
        public:
            class TimerArg
            {
            public:
                TimerArg(CB cb,void* arg,bool again):
                    m_cb_(cb),
                    m_arg_(arg),
                    m_again_(again)
                {}

                inline void 		SetID(timer_t nTimerID){
                    m_id_ = nTimerID;
                }

                void				Run()
                {
                    if(m_cb_)
                        m_cb_(m_arg_);
                    if(!m_again_)
                    {
                        timer_delete(m_id_);
                        delete this;
                    }
                }

            private:
                CB			m_cb_;
                void*		m_arg_;
                bool		m_again_;
                timer_t 	m_id_;
            };

        public:
            static bool			Init(int sig = SIGRTMAX)
            {
                s_signal_ = sig;
                struct sigaction sysact;
                sigemptyset(&sysact.sa_mask);
                sysact.sa_flags = SA_SIGINFO;
                sysact.sa_sigaction = TimerRoutine;
                return sigaction(sig, &sysact, NULL) == 0;
            }


            static void 			TimerRoutine(int signo, siginfo_t* info, void* context)
            {
                if (signo != s_signal_)
                    return;

                TimerArg* targ = static_cast<TimerArg* >(info->si_value.sival_ptr);
                targ->Run();

            }

            static bool			StartTimer(CB cb,void* arg,bool again,int nElaspe)
            {

                TimerArg* targ = new TimerArg(cb,arg,again);
                if (targ == NULL)
                    return false;

                struct sigevent evp;
                timer_t nTimerID;

                evp.sigev_notify = SIGEV_SIGNAL;
                evp.sigev_signo = s_signal_;
                evp.sigev_value.sival_ptr = static_cast<void*>(targ);

                if (timer_create(CLOCK_REALTIME, &evp, &nTimerID) == 0)
                {

                    struct itimerspec value;
                    struct itimerspec ovalue;

                    value.it_value.tv_sec = nElaspe / 1000;
                    value.it_value.tv_nsec = (nElaspe % 1000) * (1000 * 1000);

                    if (again) {
                        value.it_interval.tv_sec = value.it_value.tv_sec;
                        value.it_interval.tv_nsec = value.it_value.tv_nsec;
                    } else {
                        value.it_interval.tv_sec = 0;
                        value.it_interval.tv_nsec = 0;
                    }

                    targ->SetID(nTimerID);
                    if (timer_settime(nTimerID, 0, &value, &ovalue) == 0)
                        return true;
                    timer_delete(nTimerID);
                }
                perror("error:");
                delete targ;
                return false;
            }
        private:
            static int				s_signal_;
        };
        int Timer::s_signal_ = 0;





        void CB(void* arg)
        {
            std::cout << long(arg) << std::endl;
        }


        int main() {

            Timer::Init();
            Timer::StartTimer(CB,(void*)1,true,1000);
            Timer::StartTimer(CB,(void*)2,true,2000);
            Timer::StartTimer(CB,(void*)3,true,3000);
            while(true)
                sleep(1000);
            return 0;
        }


#参考
1. <http://linux.die.net/man/3/sigaction>
1. <http://www.kernel.org/doc/man-pages/online/pages/man2/timer_create.2.html?1351087112>
1. <http://www.kernel.org/doc/man-pages/online/pages/man2/timer_settime.2.html>
1. <http://www.kernel.org/doc/man-pages/online/pages/man2/timer_delete.2.html>

