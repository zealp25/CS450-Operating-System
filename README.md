# CS450-Operating-System

ZEAL PATEL 

A20546768

`Programming Assignment 1`

We were already provided with the code MIT xv-6 repository already had the redirect operator implemented, thus we don't have to make any changes to sh.c file.

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
The necessary changes were made to use the parallel commands where it defines the redirect and parallel commands

````
Redirect Structure

struct redircmd {
    int type;
    struct cmd *cmd;
    char *file;
    char *efile;
    int mode;
    int fd;
};

Parallel Command

struct paracmd {
    int type;
    struct cmd *left;
    struct cmd *right;
};
````

Whereas, over here parallel command has the left and right command.The redirect structure takes a command, file name, and mode in which the file should be opened. 

````
Redirect Command

struct cmd *redircmd(struct cmd *subcmd, char *file, char *efile, int mode, int fd) {
    struct redircmd *cmd;

    cmd = malloc(sizeof(*cmd));
    memset(cmd, 0, sizeof(*cmd));
    cmd->type = REDIR;
    cmd->cmd = subcmd;
    cmd->file = file;
    cmd->efile = efile;
    cmd->mode = mode;
    cmd->fd = fd;
    return (struct cmd *) cmd;
}

Parallel Command 

struct cmd *paracmd(struct cmd *left, struct cmd *right) {
    struct paracmd *cmd;

    cmd = malloc(sizeof(*cmd));
    memset(cmd, 0, sizeof(*cmd));
    cmd->type = PARA;
    cmd->left = left;
    cmd->right = right;
    return (struct cmd *) cmd;
}
````
Execution of the list commands parse resembles that of an execcmd. The program receives the left command when the symbol ';' is checked, according to the logic for parsing a list command. This left command will be carried out by the child process. We might ensure that the left command will always be executed before the right command in this way.

````
struct cmd*
parseline(char **ps, char *es)
{
  struct cmd *cmd;
  cmd = parsepipe(ps, es);

  // parse list command
  if(peek(ps, es, ";")){
    gettoken(ps, es, 0, 0);
    cmd = listcmd(cmd, parseline(ps, es));
  }
    
  if (peek(ps, es, "&")) {
    gettoken(ps, es, 0, 0);
    cmd = paracmd(cmd, parseline(ps, es));
  }
    
  return cmd;
}
````

This function's main task is to parse and execute commands. I separate the list and parallel commands. 

````
struct cmd*
parseexec(char **ps, char *es)
{
  char *q, *eq;
  int tok, argc;
  struct execcmd *cmd;
  struct cmd *ret;
  
  ret = execcmd();
  cmd = (struct execcmd*)ret;

  argc = 0;
  ret = parseredirs(ret, ps, es);
  while(!peek(ps, es, "|;&")){     // add symble "; and &"                
    if((tok=gettoken(ps, es, &q, &eq)) == 0)
      break;
    if(tok != 'a') {
      fprintf(stderr, "syntax error\n");
      exit(-1);
    }
    cmd->argv[argc] = mkcopy(q, eq);
    argc++;
    if(argc >= MAXARGS) {
      fprintf(stderr, "too many args\n");
      exit(-1);
    }
    ret = parseredirs(ret, ps, es);
    ret = parseback(ret, ps, es); // parse back commands here
  }
  cmd->argv[argc] = 0;
  return ret;
}
````

````
TEST CASE
````

| Cases | Commands | Results |
| --- | --- | --- |
| Parallel | echo a & echo b & echo c | a b c |
| Redirect Operation | echo Hello echo World echo 1 | Hello World 1 |
| Extended Operation | echo a echo b echo c echo d | a b c d |
| Illegal Input | hello | hello failed |
