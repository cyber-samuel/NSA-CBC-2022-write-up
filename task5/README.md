# Task 5
```
The FBI knew who that was, and got a warrant to seize their laptop. It looks like they had an encrypted file, which may be of use to your investigation.

We believe that the attacker may have been clever and used the same RSA key that they use for SSH to encrypt the file. We asked the FBI to take a core dump of ssh-agent that was running on the attacker's computer.

Extract the attacker's private key from the core dump, and use it to decrypt the file.

Hint: if you have the private key in PEM format, you should be able to decrypt the file with the command openssl pkeyutl -decrypt -inkey privatekey.pem -in data.enc
```

This one was rough, but only because I was not familiar with the technology in use to the degree necessary to complete the challenge.

We are given three files:
 - Core dump of ssh-agent from the attacker's computer (`core`)
 - ssh-agent binary from the attacker's computer. The computer was running Ubuntu 20.04. (`ssh-agent`)
 - Encrypted data file from the attacker's computer (`data.enc`)

Here are some links to read about the files we are given, mainly because I could spend all day talking about what we have at this point:
 - [ssh-agent](https://linux.die.net/man/1/ssh-agent)
 - [Core dumps 1](https://en.wikipedia.org/wiki/Core_dump)
 - [Core dumps 2](https://wiki.archlinux.org/title/Core_dump)
 - [Core dumps 3](https://stackoverflow.com/questions/5115613/core-dump-file-analysis)

Something else that might be useful knowledge before we start analyzing this:
```
samv in ~/hacking/codebreaker-2022/task5  file ssh-agent
ssh-agent: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=3734cf9330cd22aab10a4b215b8bcf789f0c6aeb, for GNU/Linux 3.2.0, stripped
samv in ~/hacking/codebreaker-2022/task5  file core
core: ELF 64-bit LSB core file, x86-64, version 1 (SYSV), SVR4-style, from 'ssh-agent'
```

So the `ssh-agent` binary was not compiled with debug symbols. Not surprising, but this is open-source software we're talking about!
So we could theoretically download and compile openssh with debug symbols to get a binary easier to analyze.
If you feel like doing that would help you, then by all means go for it! But today we're doing things the hard way because that's how I solved the challenge. :)

To start, I googled "how to analyze core dumps of ssh-agent"
The first link that pops up is [this](https://vnhacker.blogspot.com/2009/09/sapheads-hackjam-2009-challenge-6-or.html) which is from 2009 and describes an old, but useful way of analyzing core dumps of the ssh-agent binary. Another link that pops up is [this](https://security.humanativaspa.it/openssh-ssh-agent-shielded-private-key-extraction-x86_64-linux/), which completes the knowledge base necessary to solve the challenge using `gdb` and `Ghidra`.

The first link describes opening up the core dump file in a hex editor and looking for a `session_name` as described [here](https://github.com/openssh/openssh-portable/blob/master/ssh-agent.c#L168) in the source code for openssh. We can see that when `ssh-agent.c` is loaded into memory, the variables preceeding `session_name` are all instantiated like so:
```
typedef struct socket_entry {
	int fd;
	sock_type type;
	struct sshbuf *input;
	struct sshbuf *output;
	struct sshbuf *request;
	size_t nsession_ids;
	struct hostkey_sid *session_ids;
} SocketEntry;

u_int sockets_alloc = 0;
SocketEntry *sockets = NULL;

typedef struct identity {
	TAILQ_ENTRY(identity) next;
	struct sshkey *key;
	char *comment;
	char *provider;
	time_t death;
	u_int confirm;
	char *sk_provider;
	struct dest_constraint *dest_constraints;
	size_t ndest_constraints;
} Identity;

struct idtable {
	int nentries;
	TAILQ_HEAD(idqueue, identity) idlist;
};

/* private key table */
struct idtable *idtab;

int max_fd = 0;

/* pid of shell == parent of agent */
pid_t parent_pid = -1;
time_t parent_alive_interval = 0;

/* pid of process for which cleanup_socket is applicable */
pid_t cleanup_pid = 0;

/* pathname and directory for AUTH_SOCKET */
char socket_name[PATH_MAX];
```

The `identity` structure contains `sshkey`s loaded into memory, and `idtable` contains a list of `identity`s. Also, the variable containing a global `idtable` is `idtab`, located in memory just prior to the `socket_name`. We can calculate the offset from `session_name`, and then search the core dump with a hex editor. `pid_t` is a 32 bit integer, `'time_t` should be a 64 bit integer because `ssh-agent` is compiled for a 64 bit architecture, and `int` is a 32 bit integer. We have 2 `pid_t`s, 


