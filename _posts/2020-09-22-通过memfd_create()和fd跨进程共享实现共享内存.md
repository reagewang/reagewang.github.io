---
layout: post
title: "通过memfd_create()和fd跨进程共享实现共享内存"
tags: [共享内存]
comments: true
---

send.c
```c++
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/un.h>
#include <sys/wait.h>
#include <sys/socket.h>
#include <sys/mman.h>
#include <sys/stat.h>

#define handle_error(msg) do { perror(msg); exit(EXIT_FAILURE); } while(0)

static void send_fd(int socket, int *fds, int n)  // send fd by socket
{
    struct msghdr msg = {0};
    struct cmsghdr *cmsg;
    char buf[CMSG_SPACE(n * sizeof(int))], dup[256];
    memset(buf, '\0', sizeof(buf));
    struct iovec io = { .iov_base = &dup, .iov_len = sizeof(dup) };

    msg.msg_iov = &io;
    msg.msg_iovlen = 1;
    msg.msg_control = buf;
    msg.msg_controllen = sizeof(buf);

    cmsg = CMSG_FIRSTHDR(&msg);
    cmsg->cmsg_level = SOL_SOCKET;
    cmsg->cmsg_type = SCM_RIGHTS;
    cmsg->cmsg_len = CMSG_LEN(n * sizeof(int));

    memcpy ((int *) CMSG_DATA(cmsg), fds, n * sizeof (int));

    if (sendmsg (socket, &msg, 0) < 0) {
        handle_error ("Failed to send message");
    }
}

int main(int argc, char *argv[]) {
    int sfd, fds[2];
    struct sockaddr_un addr;

    sfd = socket(AF_UNIX, SOCK_STREAM, 0);
    if (sfd == -1) {
        handle_error ("Failed to create socket");
    }

    memset(&addr, 0, sizeof(struct sockaddr_un));
    addr.sun_family = AF_UNIX;
    strncpy(addr.sun_path, "/tmp/fd-pass.socket", sizeof(addr.sun_path) - 1);

    fds[0] = memfd_create("shma", 0);
    if (fds[0] < 0) {
        handle_error ("Failed to memfd_create shma for reading");
    } else {
        fprintf (stdout, "memfd_create fd %d in parent\n", fds[0]);
    }
    ftruncate(fds[0], SIZE);
    void *ptr0 = mmap(NULL, 0x400000, PROT_READ | PROT_WRITE, MAP_SHARED, fds[0], 0);
    memset(ptr0, 'A', 0x400000);
    munmap(ptr0, 0x400000);

    fds[1] = memfd_create("shmb", 0);
    if (fds[1] < 0) {
        handle_error ("Failed to memfd_create shmb for reading");
    } else {
        fprintf (stdout, "memfd_create fd %d in parent\n", fds[1]);
    }
    ftruncate(fds[1], 0x400000);
    void *ptr1 = mmap(NULL, 0x400000, PROT_READ | PROT_WRITE, MAP_SHARED, fds[1], 0);
    memset(ptr1, 'B', 0x400000);
    munmap(ptr1, 0x400000);

    if (connect(sfd, (struct sockaddr *) &addr, sizeof(struct sockaddr_un)) == -1) {
        handle_error ("Failed to connect to socket");
    }

    send_fd (sfd, fds, 2);

    exit(EXIT_SUCCESS);
}
```

recv.c
```c++
#include <fcntl.h>
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/un.h>
#include <sys/wait.h>
#include <sys/socket.h>

#define handle_error(msg) do { perror(msg); exit(EXIT_FAILURE); } while(0)

static int * recv_fd(int socket, int n) {
    int *fds = malloc (n * sizeof(int));
    struct msghdr msg = {0};
    struct cmsghdr *cmsg;
    char buf[CMSG_SPACE(n * sizeof(int))], dup[256];
    memset(buf, '\0', sizeof(buf));
    struct iovec io = { .iov_base = &dup, .iov_len = sizeof(dup) };

    msg.msg_iov = &io;
    msg.msg_iovlen = 1;
    msg.msg_control = buf;
    msg.msg_controllen = sizeof(buf);

    if (recvmsg (socket, &msg, 0) < 0) {
        handle_error("Failed to receive message");
    }

    cmsg = CMSG_FIRSTHDR(&msg);
    memcpy (fds, (int *) CMSG_DATA(cmsg), n * sizeof(int));
    return fds;
}

int main(int argc, char *argv[]) {
    ssize_t nbytes;
    char buffer[256];
    int sfd, cfd, *fds;
    struct sockaddr_un addr;

    sfd = socket(AF_UNIX, SOCK_STREAM, 0);
    if (sfd == -1) {
        handle_error("Failed to create socket");
    }

    if (unlink("/tmp/fd-pass.socket") == -1 && errno != ENOENT) {
        handle_error("Removing socket file failed");
    }

    memset(&addr, 0, sizeof(struct sockaddr_un));
    addr.sun_family = AF_UNIX;
    strncpy(addr.sun_path, "/tmp/fd-pass.socket", sizeof(addr.sun_path) - 1);

    if (bind(sfd, (struct sockaddr *) &addr, sizeof(struct sockaddr_un)) == -1) {
        handle_error("Failed to bind to socket");
    }

    if (listen(sfd, 5) == -1) {
        handle_error("Failed to listen on socket");
    }

    cfd = accept(sfd, NULL, NULL);
    if (cfd == -1) {
        handle_error("Failed to accept incoming connection");
    }

    fds = recv_fd (cfd, 2);

    for (int i=0; i<2; ++i) {
        fprintf (stdout, "Reading from passed fd %d\n", fds[i]);
        while ((nbytes = read(fds[i], buffer, sizeof(buffer))) > 0) {
            write(1, buffer, nbytes);
        }
        *buffer = '\0';
    }

    if (close(cfd) == -1) {
        handle_error("Failed to close client socket");
    }

    return 0;
}
```