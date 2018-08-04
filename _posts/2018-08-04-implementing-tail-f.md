---
layout: post
title:  "Implementing tail -f in Linux"
---

During implementation of utility (which purpose does not matter in context of this post) we needed to implement tail -f like behavior (i.e continuously read file updates). Traditional I/O multiplexing API (either POSIX [select(2)][select2]/[poll(2)][poll2] or Linux specific [epoll(2)][epoll2] does not work on regular files and return that file is always readable or EPERM error, depending on interface used.

In order to check for file changes we need another interface. In Linux systems we can use [inotify API][inotify7] mechanism which we can use to check for file changes. After initializing inotify instance with [inotify_init(2)][inotify_init2], we can add file to watch using [inotify_add_watch(2)][inotify_add_watch2], which returns watch descriptor. Now we can use this descriptor in I/O multiplexing API (select/poll/epoll), which will notify once descriptor is readable (i.e. there are I/O events for specified watched descriptor with associated file(s)).

Bellow is sample code which implements rudimentary tail -f behavior: only one file can be specified, code prints only changes made after executing command, code error checking is also lacking (no interruption by signal is checked, etc).

{% highlight c %}
#include <limits.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/inotify.h>
#include <sys/types.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/select.h>
#include <fcntl.h>
#include <unistd.h>
#include <errno.h>

#define BUF_LEN (10 * (sizeof(struct inotify_event) + NAME_MAX + 1))

int main(int argc, char **argv) {
    int inotify_fd, wfd, fhd, ready;
    struct inotify_event *event;
    fd_set readfds;
    ssize_t nread;
    char buf[BUF_LEN];
    char read_buf[4096];

    if (argc != 2) {
        fprintf(stderr, "Command excepts only one argument - file name to watch\n");
        exit(1);
    }
  
    FD_ZERO(&readfds);

    inotify_fd = inotify_init();
    if (inotify_fd == -1) {
        perror("inotify_init error");
        exit(1);
    }

    wfd = inotify_add_watch(inotify_fd, argv[1], IN_MODIFY);
    if (wfd == -1) {
        perror("inotify_add_watch error");
        exit (1);
    }

    FD_SET(inotify_fd, &readfds);

    fhd = open(argv[1], O_RDONLY);
    if (fhd == -1) {
        perror("open() failed");
        exit (1);
    }
    
    for (;;) {
        if (lseek (fhd, 0, SEEK_END) == -1 ) {
            perror ("lseek");
            exit(1);
        }
        ready = select(inotify_fd+1, &readfds, NULL, NULL, NULL);

        if (ready == -1) {
            perror("select()");
            exit(1);
        }
        
        nread = read(inotify_fd, buf, sizeof(buf));

        if (nread > 0) {
            event = (struct inotify_event *) buf;
            if (event->mask & IN_MODIFY) {
                while ( (nread = read (fhd, read_buf, sizeof(read_buf))) > 0 ) {
                    write(1, read_buf, nread);
                }
            }
        }
                
    }
}

{% endhighlight %}

[select2]: http://man7.org/linux/man-pages/man2/select.2.html
[poll2]: http://man7.org/linux/man-pages/man2/poll.2.html
[epoll2]: http://man7.org/linux/man-pages/man7/epoll.7.html
[inotify7]: http://man7.org/linux/man-pages/man7/inotify.7.html
[inotify_init2]: http://man7.org/linux/man-pages/man2/inotify_init.2.html
[inotify_add_watch2]: http://man7.org/linux/man-pages/man2/inotify_add_watch.2.html