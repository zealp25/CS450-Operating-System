# CS450-Operating-System

ZEAL PATEL 

A20546768

Programming Assignment 1

We were already provided with the code MIT xv-6 repository already had the redirect operator implemented, thus we don't have to make any changes to sh.c file.

Task 1 
The Shell prompt needs to be changed to CS450$

````
int getcmd(char *buf, int nbuf) {
    printf(2, "CS450$ ");
    memset(buf, 0, nbuf);
    gets(buf, nbuf);
    if (buf[0] == 0) // EOF
        return -1;
    return 0;
}
````
