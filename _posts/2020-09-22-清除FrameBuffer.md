---
layout: post
title: "清除FrameBuffer"
tags: [framebuffer]
comments: true
---

```c++
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/ioctl.h>
#include <linux/fb.h>
#include <unistd.h>

int main(void)
{
	char *fbp;
	int i,fd,ret;
	unsigned int buffersize;
	struct fb_var_screeninfo var;
	/* Open a graphics Display logical channel in blocking mode */ 
	fd = open("/dev/fb0", O_RDWR); 
	if (fd == -1) {
		perror("failed to open display device\n");
		return -1;
	}
	/* Getting varible screen information */ 
	ret = ioctl(fd, FBIOGET_VSCREENINFO, &var);
	if (ret < 0) {
		printf("FBIOGET_VSCREENINFO fail\n");
		close(fd);
		exit(0);
	}
	/* addr hold the user space address */ 
	/* Get the fix screen info */
	/* Get the variable screen information */
	buffersize = var.xres * var.yres * var.bits_per_pixel / 8; 
	fbp = (char *)mmap(NULL, buffersize, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
	for (i=0; i<buffersize; i++) {
		*(fbp + i) = 0;
    }
	/* closing of channels */ 
	munmap(fbp, buffersize);
	close (fd);
	return 0;
}
```