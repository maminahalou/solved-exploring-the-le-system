Download Link: https://assignmentchef.com/product/solved-exploring-the-le-system
<br>
<h1>                  1        Introduction</h1>

This is a quite easy exercise but you will learn a lot about how les are represented. We will not look to the actual content of the les but on the so call meta-data that we have of each le. In order to understand how much data there is, we will implement a command that you have probably used many times in the shell. We will also gather some statistics on the size of les and present the numbers in some nice graphs.

<h1>                  2         List a directory</h1>

The command that we will implement is a version of the well known ls command, the command you use to list the content of a directory. Our rst try will not be very impressive but at least it will do something. Let’s call it myls so we can use the regular ls to verify that we do the right thing. Create a le myls.c and start coding.

<h2>                  2.1        read the directory</h2>

To our help we have a library call opendir() that will make things a bit easier for us. This procedure will return a pointer to a directory stream i.e. a sequence of directory entries. We can then access each of these entries using the library procedure readdir(). You should by now be able to gure out what header les you need to include, so we will not list them in the code sections.

int main( int               argc ,         char ∗argv [ ] )         {

if (            argc &lt; 2 ) {

perror (“usage :             myls &lt;dir &gt;
” );

return −1;

}

char ∗path = argv [ 1 ] ; DIR ∗dirp = opendir ( path ); struct dirent ∗entry ;

while (( entry = readdir ( dirp )) != NULL) { printf (“type : %u” , entry−&gt;d_type ); printf (“tinode %lu” , entry−&gt;d_ino );

printf (“tname : %s
” ,              entry−&gt;d_name);

}

}

Do man readddir and you will see what the dirent structure looks like and what we could nd more than what we have printed (not much). If you compile and run the program you will see your rst attempt of mimicking ls, it works but we’re far from there. This is what my attempt looked like.

<table>

 <tbody>

  <tr>

   <td width="99"></td>

  </tr>

  <tr>

   <td></td>

   <td></td>

  </tr>

 </tbody>

</table>

&gt; gcc -o myls myls.c

&gt; ./myls ./ type: 4        inode: 917888 name: . type: 8     inode: 945912 name: myls type: 8        inode: 963825 name: myls.c type              4:              inode: 940418 name: ..

So what do we have here: a type, the inode number and the name. We can make it a bit more readable by interpreting the type information.

<h2>                  2.2       the type</h2>

The interpretation is found in the man pages for readdir() and we can print them out using a switch statement.

<table width="422">

 <tbody>

  <tr>

   <td colspan="3" width="260">while (( entry = readdir ( dirp ))</td>

   <td rowspan="2" width="162">!= NULL) {</td>

  </tr>

  <tr>

   <td colspan="2" width="236">        switch(                     entry−&gt;d_type) {</td>

   <td width="24"> </td>

  </tr>

  <tr>

   <td width="174">               case DT_BLK         :printf (“b : ” ); break ;</td>

   <td width="62">// This</td>

   <td width="24">is</td>

   <td width="162">a        block       device .</td>

  </tr>

  <tr>

   <td width="174">               case DT_CHR         :printf (“c : ” ); break ;</td>

   <td width="62">//This</td>

   <td width="24">is</td>

   <td width="162">a       character          device .</td>

  </tr>

  <tr>

   <td width="174">               case DT_DIR          :printf (“d : ” ); break ;</td>

   <td width="62">//This</td>

   <td width="24">is</td>

   <td width="162">a       directory .</td>

  </tr>

  <tr>

   <td width="174">case DT_FIFO : printf (“p : ” ); break ;</td>

   <td width="62">//This</td>

   <td width="24">is</td>

   <td width="162">a named pipe           .</td>

  </tr>

  <tr>

   <td width="174">               case DT_LNK         :printf (” l : ” ); break ;</td>

   <td width="62">//This</td>

   <td width="24">is</td>

   <td width="162">a      symbolic        link .</td>

  </tr>

  <tr>

   <td width="174">               case DT_REG         :</td>

   <td width="62">//This</td>

   <td width="24">is</td>

   <td width="162">a      regular       f i l e .</td>

  </tr>

 </tbody>

</table>

printf (” f : ” );

break ; case DT_SOCK : //This is a UNIX domain socket . printf (“s : ” );

break ; case DT_UNKNOWN :              // The   f i l e      type       is             unknown . printf (“u : ” );

break ;

}

printf (“tinode %lu” ,                   entry−&gt;d_ino );

printf (“tname : %s
” ,              entry−&gt;d_name);

}

As seen in the code above we are not talking about the type in terms of pdf, txt or a C source le. The type we are talking about is to di erentiate directories and symbolic links etc from regular les. To nd out if a particular le is a pdf le we would have to look at its name and see if ends in .pdf; this is only a convention and you’re of course free to name a pdf- le to foo.jpg if you want to; doing so will however make it hard to automatically determine which application to open when you want to view the content of the le.

This is all there is to the directory, it’s a mapping from names to inodes. We do not now anything about the le more than the name and remember that the name is just something that is valid in the directory that we are currently looking at.

<h2>                  2.3        more information</h2>

To nd more information about a particular le we use the system call fstatat(). This procedure will populate a stat structure with all the information about the le we are looking for. There is also a stat() procedure but this procedure would look-up a le name in the current directory which will probably not be the directory that we are looking at. The fstatat() procedure allows us to specify in which directory we should do the look-up. struct stat file_st ; fstatat ( dirfd ( dirp ) , entry−&gt;d_name, &amp;file_st , 0);

If we insert this above the printf() statement we can write out more information about the le, for example its size.

printf (“tinode : %lu” , entry−&gt;d_ino ); printf (“ tsize : %lu” , file_st . st_size ); printf (“tname : %s
” , entry−&gt;d_name);

Take a look in the man pages of fstatat() and you will nd more information about the le.

<h1>                  3           Things are not that simple</h1>

To disturb your world of comfort, where directories map names to inodes and inodes are something that is on some disk we will complicate things a bit.

First we will create a dummy directory called dram and then mount a tmpfs le system using the directory as a mount point. Let’s rst create a directory and see what it looks like.

&gt; mkdir foo :

&gt; ./myls .

:

As you see the directory is given a new inode number and we can verify that we do the right thing by looking at the output of the regular ls command.

&gt; ls -il .

<table>

 <tbody>

  <tr>

   <td width="99"></td>

  </tr>

  <tr>

   <td></td>

   <td></td>

  </tr>

 </tbody>

</table>

:

Now let’s try this – mount a tmpfs le system using the dram directory as the destination.

&gt; sudo mount -t tmpfs tmpfs ./dram :

Nothing much has changed and this can be veri ed my looking at the output from myls. But check what you see when you try the regular command ls -li – hmm, what is going on?

If you wan to get even more confused look at the output when we look inside the dram directory.

&gt; ./myls dram :

&gt; ls -ila dram

:

The regular ls command does not report what we see with myls, but instead collects the information from the stat structure. Try the following and things might be a bit more clear.

<table width="342">

 <tbody>

  <tr>

   <td width="199">printf (“tinode : %lu” ,</td>

   <td width="144">entry−&gt;d_ino );</td>

  </tr>

  <tr>

   <td width="199">printf (“tdev :                    0x%lx ” ,</td>

   <td width="144">file_st . st_dev );</td>

  </tr>

  <tr>

   <td width="199">printf (“tinode : %lu” ,</td>

   <td width="144">file_st . ino );</td>

  </tr>

  <tr>

   <td width="199">printf (“ tsize : %lu” ,</td>

   <td width="144">file_st . st_size );</td>

  </tr>

  <tr>

   <td width="199">printf (“tname : %s
” ,</td>

   <td width="144">entry−&gt;d_name);</td>

  </tr>

 </tbody>

</table>




As you see ls uses the inode number found in the stat structure rather than in the directory listing. Also note that the inode numbers of the mounted directory, 2, is the same as in your regular root directory. If you try the command df you will see a list of all le systems currently mounted in your system and if do some more investigation you will nd that all of them have a root directory with inode number 2.

Inode numbers are local to a le system and the operating system needs to keep track of which le system we are talking about; or, what device the le system is found at. The hierarchical name space were directories serves as mount points for di erent le system, provides a seamless name space. You’re normally not aware of which le system that is activated. If you’re regular Windows user you might be used to the driver letters C:, D: etc, these are the Windows equivalent of mounted le systems. You might wonder where A: and B: went but if you attach a floppy disk drive you might have it mounted as the A: drive.

<h2>                   3.1       the device</h2>

We printed the st_dev value in hex and we did so for a reason. The value that was printed, let’s assume 0x801, is interpreted as disk 8, partition 1. If you examine the /dev directory you probably see that the le system that it is referring to is your main disk drive. This might be di erent depending on which machine you run but you should be able to work out the details.

&gt; ls -l /dev/sda*

So the stat structure contains a local inode number and which device this inode node number belongs to. This indirection from the true inodes that are on disk is called virtual inodes or vnodes; an abstraction layer that allows your Unix le system to present several di erent le systems with the same interface.

Enough about mounted le systems. If you unmount dram we shall try to gather some statistics of your les on your machine.

<h1>                   4        Traverse the tree</h1>

Let’s write a program that counts the number of les in a directory including all sub-directories. We have most of the components we need and if we can only avoid the most obvious pit-falls we should be done i ten minutes. Create a new le total.c and start coding.

<h2>                   4.1         a recursive solution</h2>

Since we know that the directory tree is not very deep we will implement a recursive procedure. The procedure count() will open a directory, count the number of les and recursively count the number of les in its sub-directories. We do a quick and dirty implementation where we assume that no directory path is longer than 1024 characters – a proper solution would allocate the required amount of memory on the heap.

unsigned long count (char ∗path ) { unsigned long total = 0; DIR ∗dirp = opendir ( path ); char subdir [1024]; struct dirent ∗entry ; struct stat file_st ;

while (( entry = readdir ( dirp )) != NULL) { switch( entry−&gt;d_type) {

<table width="413">

 <tbody>

  <tr>

   <td width="227">case DT_DIR:                 //This       is        a:</td>

   <td colspan="2" width="186">directory .</td>

  </tr>

  <tr>

   <td width="227">sprintf ( subdir , “%s/%s” , total += count ( subdir ); break ;</td>

   <td width="60">path ,</td>

   <td width="126">entry−&gt;d_name);</td>

  </tr>

  <tr>

   <td width="227">case DT_REG:    //This    is             a total++; break ;</td>

   <td width="60">regular</td>

   <td width="126">f i l e .</td>

  </tr>

 </tbody>

</table>

default : break ;

}

} closedir ( dirp );

return        total ;

}

We have to think a bit here – the directory contains two very special entries “..” and “.” . If we follow these paths we will for sure wait for a very long time before we receive any results. We need to prevent the count() procedure from going into an endless loop so we insert the following check. if (( strcmp ( entry−&gt;d_name, ” . “) == 0)            |              ( strcmp ( entry−&gt;d_name,          ” . . “) == 0))        {

break ; };

A small main() procedure and we’re done. If you have added all the right include directives you should be able to compile and run your le counter.

int main( int                argc ,         char ∗argv [ ] )        {

if (            argc &lt; 2 ) {

perror (“usage :               total &lt;dir &gt;
” );

return −1;

}

char ∗path = argv [ 1 ] ; unsigned long total = count ( path );

printf (“The            directory %s             contains %lu          f i l e s 
” ,     path ,        total );

}

<h2>                   4.2        exceeding you rights</h2>

If you try to run you program on for example /etc you will probably get the a segmentation fault. Instead of xing this directly (I know the reason) we can try to use gdb to try to nd out what happened.

./total /etc segmentation fault (core dumped)

We rst compile the program with the -g ag set. This will produce a binary with some additional debug information that will allow us to use gdb.

&gt; gcc -g -o total total.c

We then start gdb giving the program as an argument.

&gt; gdb total : :

We now have a (gdb) prompt and can start using the debugger. In this small example we will only run the program, let it crash and then try to gure out what happened. You start the program with the run command, giving /etc as an argument. It will look something like the following.

(gdb) run /etc

Starting program: …../src/total /etc

Program received signal SIGSEGV, Segmentation fault.

0x00007ffff7ad58a2 in __readdir (dirp=0x0) at ../sysdeps/posix/readdir.c:44 44 ../sysdeps/posix/readdir.c: No such file or directory.

(gdb)

This tells us that the segmentation fault occurred in the system call __readdir where dirp was a null pointer. To see where in our code this happened we step up in the call stack.

(gdb) up

#1                        0x000000000040084e in count (path=0x7fffffffd9f0 “/etc/…”) at total.c:66

66                   while((entry = readdir(dirp)) != NULL) {

(gdb)

Now we see were we are in our code – the while statement in the count() procedure. We can con rm that dirp is actually a null pointer by printing its value.

(gdb) p dirp

$1 = (DIR *) 0x0

The question is why; we received the pointer from the opendir() procedure so the question is what value path had when we called it. Printing the value of path will (in my case) reveal that something strange happened when we tried to open “/etc/polkit-1/localauthority”.

(gdb) p path

$2 = 0x7fffffffd9f0 “/etc/polkit-1/localauthority”

(gdb)

If you look at the directory that caused your problem (if there was one) you might              nd the reason for the failure.

&gt; ls -ld /etc/polkit-1/localauthority drwx——    7 root    root       4096 2016-04-21 00:11 /etc/polkit-1/localauthority

Hmmm, owned by root with the access rights “rwx “. No wonder a regular user could not read that directory. Let’s x our code so we take care of the case where we, for some reason, will not be able to read the directory. if ( dirp == NULL) {

printf (“not         able     to      open %s
” ,        path );

return       0;

}

Ok, give it a try.

<h2>                   4.3        double counting</h2>

What we’re counting is the number of links to les, if a le object is linked to by several links this object will be counted twice. We could of course keep track of which les that we have seen and make sure that we only count them twice. If you tried to implemented this, how would you identify unique les? Would the inode number be enough?

5        How large are        les?

If we forget about the problem with double counting, we can implement a program that generates some statistics of le sizes. We start with some assumptions and a global data structure where will store the result. Create a le called freq.c, or rather take copy of total.c since we’re going to reuse most of the code.

A frequency table will keep track of how many les we have of certain sizes. We’re not particularly interested in the exact distribution so we only keep track of the size in steps of power of two. We do not count les of size 0 and only keep track of les up to the size 2<sup>FREQ</sup><sup>_</sup><sup>MAX</sup>. The last entry in the table will contain all larger les and if the machine that you’re running on is not also the machine that keeps all your movies, I guess 2<sup>32 </sup>will be ne for our purposes.

#define FREQ_MAX 32 unsigned long freq [FREQ_MAX] ;

void add_to_freq(unsigned long                               size ) {

if ( size != 0) { int j = 0; int n = 2; while( size / n != 0 &amp; j &lt; FREQ_MAX) {

n = 2∗n ; j++; } freq [ j ]++;

}

}

We now change the procedure count() so that it will call add_to_freq() with the size of each le that it sees. We don’t have to return anything so there are only small changes. We check that fstatat() succeeds before using file_st, it could be a le that we do not have read permission to.

void count (char ∗path ) {

:

case DT_REG:             //This       is     a      regular       f i l e .

if ( fstatat ( dirfd ( dirp ) ,                    entry−&gt;d_name, &amp;file_st ,                 0) == 0) {

add_to_freq( file_st . st_size );

} break ;

:

}

Almost done, some small changes to the main() procedure and we’re done.

:

printf (“#The directory %s : number of f i l e s smaller than 2^k:
” , path ); printf (“#ktnumber
” );

for ( int        j= 0;          j &lt; FREQ_MAX;          j++) {

printf (“%dt%lu
” ,            ( j +1),        freq [ j ] ) ;

}

:

Hmm, should work – try with a smaller directory rst and then gather some statistics of a larger directory (try /usr). If everything works we should have nice printout of a table and we can of course not resist to explore this data using gnuplot.

<h1>                   6         Some nice graphs</h1>

Start by saving the histogram in a le called freq.dat and then you can generate your rst graph in one line of code.

&gt;./freq /usr &gt; freq.dat

&gt; gnuplot :

: gnuplot&gt; plot “freq.dat” using 1:2 with boxes

If you graph looks anything like mine you see that most le are between 1K and 2K bytes (in the 2<sup>11 </sup>bucket). There are plenty of smaller les but few that are smaller than 32 bytes; les above one megabyte are also rare. It looks like half of the les are between 512 and 4<em>K </em>bytes.

An alternative way of presenting these numbers is as a cumulative frequency diagram. Try the following:

gnuplot&gt; a=0

gnuplot&gt; cumulative_freq(x)=(a=a+x,a) : gnuplot&gt; plot “freq.dat” u 1:(cumulative_freq($2)) w linespoints

This diagram adds the frequencies as we go and shows how many les are less than a certain size. If we know that there were 280000 le in the /usr directory we could get a nice y-axis that would give us the percentage of all les.

gnuplot&gt; plot “freq.dat” u 1:(cumulative_freq($2)/280000) w linespoints

You could of course wonder how much of the hard drive is taken up by les of what size and we can give a hint of this by multiplying the frequency with the average size of the category. If we adapt the function to take the size into account we have the following:

gnuplot&gt; a=0

gnuplot&gt; cumulative_size(k, x)=(a=a+(x*((2**k)-((2**(k-1)))/2)),a) : gnuplot&gt; plot “freq.dat” u 1:(cumulative_size($1,$2)) w linespoints

The graphs that we have generated now will give you a quick overview of what the le system looks like. If you would use them in a presentation you would of course do some more work to x the scales, titles etc. The graph should contain all information needed to interpret it; it should not be open for guessing what the x-axis really means.

<h1>                   7       Summary</h1>

So with some simple coding I hope that you have learned some more about the le system and even if it was nothing completely new, at least it gave you some rst hand experience of what it looks like. It’s one thing to look at the power-point slide, another actually doing it.