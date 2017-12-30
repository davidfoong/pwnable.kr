# pwnable.kr
Just some of my thought process going through the Toddler's Bottle beginner levels.

1: [fd]

Looking at the program, I see the program is looking for an input of LETMEWIN to get the flag. However, you can't just enter it as an argument.
The clue asks you what a file descriptor is. After googling, I learned that there are three file descriptors. Standard input is accessed at integer value 0,
so I looked at the line :
int fd = atoi( argv[1] ) - 0x1234;
took 0x1234, converted it to decimal, and entered in 4660 as the file argument. This allowed me to write to STDIN, and I entered LETMEWIN.
the flag was returned.


2: collision

Looking at the code for col.c, we see a few restrictions for our argument. The passcode length must be length 20.
The password you enter is taken into chunks of four and totalled. Therefore, we need 5 integers whose total should be equal to 0x21DD09EC. 

```c
col@ubuntu:~$ python
Python 2.7.12 (default, Jul  1 2016, 15:12:24)
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 0x21DD09EC
568134124
>>>
```
Next, I converted 0x21DD09EC into decimal, which is: 568134124. We can see by looking at it that 568134124 is not cleanly divisible by 5.
I plugged it into a calculator and got 113626828.8, which is 113626828 and 4/5. This means that if we round off 113626828.8 to 113626828 and subtract 4, we 
have 113626824 * 4 + 113626828 = 568134124. So we have to enter those in hex. I used python:

```c
./col `python -c 'print "\xc8\xce\xc5\x06"*4 + "\xcc\xce\xc5\x06"'`
```
The flag was returned!

3: mistake

This one relied used some knowledge from the first one I did, fd. 
```c
mistake@ubuntu:~$ cat mistake.c
#include <stdio.h>
#include <fcntl.h>

#define PW_LEN 10
#define XORKEY 1

void xor(char* s, int len){
        int i;
        for(i=0; i<len; i++){
                s[i] ^= XORKEY;
        }
}

int main(int argc, char* argv[]){

        int fd;
        if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0){
                printf("can't open password %d\n", fd);
                return 0;
        }

        printf("do not bruteforce...\n");
        sleep(time(0)%20);

        char pw_buf[PW_LEN+1];
        int len;
        if(!(len=read(fd,pw_buf,PW_LEN) > 0)){
                printf("read error\n");
                close(fd);
                return 0;
        }

        char pw_buf2[PW_LEN+1];
        printf("input password : ");
        scanf("%10s", pw_buf2);

        // xor your input
        xor(pw_buf2, 10);

        if(!strncmp(pw_buf, pw_buf2, PW_LEN)){
                printf("Password OK\n");
                system("/bin/cat flag\n");
        }
        else{
                printf("Wrong Password\n");
        }

        close(fd);
        return 0;
}
```
From 
