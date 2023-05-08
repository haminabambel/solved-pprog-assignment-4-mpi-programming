Download Link: https://assignmentchef.com/product/solved-pprog-assignment-4-mpi-programming
<br>
5/5 - (1 vote)

<main>







</main>

The purpose of this assignment is to familiarize yourself with MPI programming.




<hr>

Get the source code:

<pre><code class="shell language-shell">$ wget https://nycu-sslab.github.io/PP-f21/HW4/HW4.zip$ unzip HW4.zip -d HW4$ cd HW4</code></pre>

<h2 id="0settinguptheenvironment">0. Setting Up the Environment</h2>

To make MPI work properly, you need to be able to execute jobs on remote nodes without typing a password. You will need to generate an ssh key by yourself. You may follow the instructions below or refer to <a href="http://linuxproblem.org/art_9.html" target="_blank" rel="noopener">this article</a> for instruction.

You will also need to configure your <code>~/.ssh/config</code> file to define host names. Use <code>HW4/config</code> as a template for the configuration. Notice that in the configuration template, the string follows each <code>User</code> should be your student ID.

<strong>ATTENTION: We will test your code based on this config.</strong>

<pre><code class="shell language-shell"># &lt;login to one of the PP nodes&gt;$ mkdir -p ~/.ssh$ ssh-keygen -t rsa # Leave all empty# &lt;setup the config&gt;$ cat ~/.ssh/id_rsa.pub &gt;&gt; ~/.ssh/authorized_keys</code></pre>

Wait minutes for system setup… Then, login to all machines at least once to make sure the configuration works. This will save the SSH fingerprints of the machines in <code>~/.ssh/known_hosts</code>.

<pre><code class="shell language-shell"># Log in without entering password$ ssh pp1$ ssh pp2$ ssh pp3$ ssh pp4$ ...$ ssh pp9</code></pre>



<h2 id="1part1gettingfamiliarwithmpiprogramming">1. Part 1: Getting Familiar with MPI Programming</h2>

<h3 id="11mpihelloworldgettingfamiliarwiththempienvironment">1.1 MPI Hello World, Getting Familiar with the MPI Environment</h3>

Here we’re going to implement our first MPI program.

Expected knowledge includes basic understanding of the MPI environment, how to compile an MPI program, how to set the number of MPI processes, and retrieve the rank of the process and number of MPI processes at runtime.

Let’s learn how to launch an MPI code on NYCU-PP workstations.

There is a starter code for you. Look at <code>hello.cc</code>. Please finish the <em>TODO</em>s with the <code>MPI_Comm_size</code> and <code>MPI_Comm_rank</code> functions.

We will work on NFS (which is already set up ), so we don’t need to copy and compile the code again and again on different nodes.

Let’s first compile <code>hello.cc</code> on NFS:

<pre><code class="shell language-shell">$ cd ./HW4$ mpicxx ./hello.cc -o mpi_hello</code></pre>

Then create a file <code>hosts</code>, which is a text file with hosts specified, one per line. We will use the file to specify on which hosts to launch MPI processes when running <code>mpirun</code>. For example, we put the following two lines into the file to specify we will launch MPI processes on nodes <code>pp1</code> and <code>pp2</code>. Notice that you should use the same hostnames as in <code>~/.ssh/config</code>.

<code>hosts</code>:

<pre><code class="config language-config">pp1 # 1st node hostnamepp2 # 2ed node hostname</code></pre>

Make sure <code>mpi_hello</code> is located in the same directory hierarchy on the two nodes (should not be a problem under NFS) before running <code>mpirun</code> to start MPI processes:

<pre><code class="shell language-shell">$ mpirun -np 8 --hostfile hosts mpi_hello</code></pre>

This command will launch eight processes to run <code>mpi_hello</code> on the two nodes, and you should be able to get an output similar to:

<pre><code class="log language-log">Hello world from processor ec037-072, rank 0 out of 8 processorsHello world from processor ec037-072, rank 1 out of 8 processorsHello world from processor ec037-072, rank 3 out of 8 processorsHello world from processor ec037-072, rank 2 out of 8 processorsHello world from processor ec037-073, rank 4 out of 8 processorsHello world from processor ec037-073, rank 6 out of 8 processorsHello world from processor ec037-073, rank 5 out of 8 processorsHello world from processor ec037-073, rank 7 out of 8 processors</code></pre>

PS: There may be some <code>No protocol specified</code> messages in the beginning, it’s ok to just ignore them.

<strong>Code Implementation</strong>: (<em>3 points</em>)

<ul>

 <li>Complete <code>hello.cc</code>: the Hello World code in C.</li>

 <li><code>mpirun -np 8 --hostfile hosts mpi_hello</code> should run correctly</li>

 <li>The argument of <code>-np</code> is expected to be any positive integer less than or equal to 16.</li>

</ul>

<strong>Q1</strong> (<em>3 points</em>)

<ol>

 <li>How do you control the number of MPI processes on each node? (<em>1.5 points</em>)</li>

 <li>Which functions do you use for retrieving the rank of an MPI process and the total number of processes? (<em>1.5 points</em>)</li>

</ol>

<h3 id="12calculatingpiwithmpi">1.2 Calculating PI with MPI</h3>

In this exercise, we are going to parallelize the calculation of Pi following a Monte Carlo method and using different communication functions and measure their performance.

Expected knowledge is MPI blocking and non-blocking communication, collective operations, and one-sided communication.

<strong>Instructions</strong>: Write different versions of MPI codes to calculate Pi. The technique we are implementing relies on random sampling to obtain numerical results. You have already implemented this in the previous assignments.

<h4 id="121mpiblockingcommunicationlinearreductionalgorithm">1.2.1 MPI Blocking Communication &amp; Linear Reduction Algorithm</h4>

In our parallel implementation, we split the number of iterations of the main loop into all the processes (i.e., <code>NUM_ITER / num_ranks</code>). Each process will calculate an intermediate <code>count</code>, which is going to be used afterward to calculate the final value of Pi.

To calculate Pi, you need to send all the intermediate counts of each process to rank 0. This communication algorithm for reduction operation is called linear as the communication costs scales as the number of processes.

An example of linear reduction with eight processes is as follows:

<img decoding="async" alt="image" data-recalc-dims="1" data-src="https://i0.wp.com/user-images.githubusercontent.com/18013815/96628318-aa40a800-1344-11eb-9d79-eff7988c2f15.png?w=980&amp;ssl=1" class="lazyload" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==">

 <noscript>

  <img decoding="async" src="https://i0.wp.com/user-images.githubusercontent.com/18013815/96628318-aa40a800-1344-11eb-9d79-eff7988c2f15.png?w=980&amp;ssl=1" alt="image" data-recalc-dims="1">

 </noscript>

Rank 0 is going to receive the intermediate <code>count</code> calculated by each process and accumulate them to its own count value. (Rank 0 also needs to calculate a <code>count</code>, and this stands for the following sections in Part One.) Finally, after rank 0 has received and accumulated all the intermediate counts, it will calculate Pi and show the result, as in the original code.

Implement this code using blocking communication and test its performance.

<blockquote>

 <em>Hint</em> 1. Change the main loop to include the number of iterations per process, and not <code>NUM_ITER</code> (which is the total number of iterations).

 <em>Hint</em> 2. Do not forget to multiply the seed of <code>srand()</code> with the rank of each process (e.g., “rank * SEED”) to ensure the RNGs of processes generate different random numbers. There is a starter code <code>pi_block_linear.cc</code> for you.

</blockquote>

<pre><code class="shell language-shell">$ mpicxx pi_block_linear.cc -o pi_block_linear; mpirun -np 4 --hostfile hosts pi_block_linear 1000000000</code></pre>

<strong>Code Implementation</strong>: (<em>7 points</em>)

<ul>

 <li>Complete <code>pi_block_linear.cc</code>: the MPI block communication for PI.</li>

 <li>Implement the code using a <strong>linear algorithm</strong> with <code>MPI_Send</code>/<code>MPI_Recv</code>.</li>

 <li>Do not modify the output messages.</li>

 <li><code>$ mpicxx pi_block_linear.cc -o pi_block_linear; mpirun -np 4 --hostfile hosts pi_block_linear 1000000000</code> should run correctly.</li>

 <li>The argument of <code>-np</code> is expected to be any positive integer less than or equal to 16.</li>

 <li><code>mpirun -np 4 --hostfile hosts pi_block_linear 1000000000</code> should run almost as fast as TA’s reference version (refer to <code>/HW4/ref</code>). The difference (in execution time) should be within 15%.</li>

</ul>

<blockquote>

 You may use up to five nodes (and up to four processors in each node) for Part 1, so you may want to try different configurations in the hostfile. <strong>Q2</strong> (<em>2 points</em>)

 <ol>

  <li>Why <code>MPI_Send</code> and <code>MPI_Recv</code> are called “blocking” communication? (<em>1 points</em>)</li>

  <li>Measure the performance (execution time) of the code for 2, 4, 8, 12, 16 MPI processes and plot it. (<em>1 points</em>)</li>

 </ol>

</blockquote>

<h4 id="122mpiblockingcommunicationbinarytreereductioncommunicationalgorithm">1.2.2 MPI Blocking Communication &amp; Binary Tree Reduction Communication Algorithm</h4>

Implement the binary tree communication algorithm for performing the reduction on rank 0 using blocking communication (e.g., <code>MPI_Send</code>/<code>MPI_Recv</code>).

The communication pattern for a reduction with a binary tree with eight processes is as follows:

<img decoding="async" alt="image" data-recalc-dims="1" data-src="https://i0.wp.com/user-images.githubusercontent.com/18013815/96628491-e116be00-1344-11eb-94b4-2b19a62fa016.png?w=980&amp;ssl=1" class="lazyload" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==">

 <noscript>

  <img decoding="async" src="https://i0.wp.com/user-images.githubusercontent.com/18013815/96628491-e116be00-1344-11eb-94b4-2b19a62fa016.png?w=980&amp;ssl=1" alt="image" data-recalc-dims="1">

 </noscript>

In you implementation, you can assume that we use a power-of-two number of processes.

There is a starter code <code>pi_block_tree.cc</code> for you.

<pre><code class="shell language-shell">$ mpicxx pi_block_tree.cc -o pi_block_tree; mpirun -np 4 --hostfile hosts pi_block_tree 1000000000</code></pre>

<strong>Code Implementation</strong>: (<em>7 points</em>)

<ul>

 <li>Complete <code>pi_block_tree.cc</code>: the MPI block communication for PI.</li>

 <li>Implement the code using a <strong>binary tree reduction algorithm</strong> with <code>MPI_Send</code>/<code>MPI_Recv</code>.</li>

 <li>Do not modify the output messages.</li>

 <li><code>$ mpicxx pi_block_tree.cc -o pi_block_tree; mpirun -np 4 --hostfile hosts pi_block_tree 1000000000</code> should run correctly.</li>

 <li>The argument of <code>-np</code> is expected to be any power-of-two number less than or equal to 16.</li>

 <li><code>mpirun -np 4 --hostfile hosts pi_block_tree 1000000000</code> should run almost as fast as TA’s reference version. The difference (in execution time) should be within 15%.</li>

</ul>

<strong>Q3</strong> (<em>6 points</em>)

<ol>

 <li>Measure the performance (execution time) of the code for 2, 4, 8, 16 MPI processes and plot it. (<em>1 points</em>)</li>

 <li>How does the performance of binary tree reduction compare to the performance of linear reduction? (<em>2 points</em>)</li>

 <li>Increasing the number of processes, which approach (linear/tree) is going to perform better? Why? Think about the number of messages and their costs. (<em>3 points</em>)</li>

</ol>

<h4 id="123mpinonblockingcommunicationlinearreductionalgorithm">1.2.3 MPI Non-Blocking Communication &amp; Linear Reduction Algorithm</h4>

Use non-blocking communication for the linear reduction operation (in Section <a href="#121--mpi-blocking-communication--linear-reduction-algorithm">1.2.1</a>).

<blockquote>

 <em>Hint</em>: Use a non-blocking <code>MPI_Irecv()</code> (MPI Receive with Immediate return). The basic idea is that rank 0 is going to issue all the receive operations and then wait for them to finish. You can either use <code>MPI_Wait()</code> individually to wait for each request to finish or <code>MPI_Waitall()</code>. Regardless of your decision, keep in mind that we want you to perform the receive operations in parallel. Thus, do not call <code>MPI_Irecv()</code> and immediately <code>MPI_Wait()</code>! In addition, we recommend you allocate an array of <code>MPI_Request</code> and also an array of counts (i.e., one for each receive needed). There is a starter code <code>pi_nonblock_linear.cc</code> for you.

</blockquote>

<pre><code class="shell language-shell">$ mpicxx pi_nonblock_linear.cc -o pi_nonblock_linear; mpirun -np 4 --hostfile hosts pi_nonblock_linear 1000000000</code></pre>

<strong>Code Implementation</strong>: (<em>7 points</em>)

<ul>

 <li>Complete <code>pi_nonblock_linear.cc</code>: the MPI non-blocking communication for PI.</li>

 <li>Implement the code using a <strong>non-blocking algorithm</strong> with <code>MPI_Send</code>/<code>MPI_IRecv</code>/<code>MPI_Wait</code>/<code>MPI_Waitall</code>.</li>

 <li>Do not modify the output messages.</li>

 <li><code>$ mpicxx pi_nonblock_linear.cc -o pi_nonblock_linear; mpirun -np 4 --hostfile hosts pi_nonblock_linear 1000000000</code> should run correctly.</li>

 <li>The argument of <code>-np</code> is expected to be any positive integer less than or equal to 16.</li>

 <li><code>mpirun -np 4 --hostfile hosts pi_nonblock_linear 1000000000</code> should run almost as fast as TA’s reference version. The difference (in execution time) should be within 15%.</li>

</ul>

<strong>Q4</strong> (<em>5 points</em>)

<ol>

 <li>Measure the performance (execution time) of the code for 2, 4, 8, 12, 16 MPI processes and plot it. (<em>1 points</em>)</li>

 <li>What are the MPI functions for non-blocking communication? (<em>1 points</em>)</li>

 <li>How the performance of non-blocking communication compares to the performance of blocking communication? (<em>3 points</em>)</li>

</ol>

<h4 id="124mpicollectivempi_gather">1.2.4 MPI Collective: MPI_Gather</h4>

Use the collective <code>MPI_Gather()</code> operation, instead of point-to-point communication.

<em>Hint</em>: You can keep rank 0 as the root of the communication and still make this process aggregate manually the intermediate counts. Remember that the goal of <code>MPI_Gather()</code> is to provide the root with an array of all the intermediate values. Reuse the array of counts as the output for the gather operation.

There is a starter code <code>pi_gather.cc</code> for you.

<pre><code class="shell language-shell">$ mpicxx pi_gather.cc -o pi_gather; mpirun -np 4 --hostfile hosts pi_gather 1000000000</code></pre>

<strong>Code Implementation</strong>: (<em>7 points</em>)

<ul>

 <li>Complete <code>pi_gather.cc</code>: the MPI gather communication for PI.</li>

 <li>Implement the code using <code>MPI_Gather</code>.</li>

 <li>Do not modify the output messages.</li>

 <li><code>$ mpicxx pi_gather.cc -o pi_gather; mpirun -np 4 --hostfile hosts pi_gather 1000000000</code> should run correctly.</li>

 <li>The argument of <code>-np</code> is expected to be any positive integer less than or equal to 16.</li>

 <li><code>mpirun -np 4 --hostfile hosts pi_gather 1000000000</code> should run almost as fast as TA’s reference version. The difference (in execution time) should be within 15%.</li>

</ul>

<strong>Q5</strong> (<em>1 points</em>)

<ol>

 <li>Measure the performance (execution time) of the code for 2, 4, 8, 12, 16 MPI processes and plot it. (<em>1 points</em>)</li>

</ol>

<h4 id="125mpicollectivempi_reduce">1.2.5 MPI Collective: MPI_Reduce</h4>

Use the collective <code>MPI_Reduce()</code> operation.

<blockquote>

 <em>Hint</em> 1: Remember that the goal of <code>MPI_Reduce()</code> is to perform a collective computation. Use the <code>MPI_SUM</code> operator to aggregate all the intermediate count values into rank 0, But, watch out: rank 0 has to provide its own count as well, alongside the one from the other processes.

 <em>Hint</em> 2: The send buffer of <code>MPI_Reduce()</code> must not match the receive buffer. In other words, use a different variable on rank 0 to store the result. There is a starter code <code>pi_reduce.cc</code> for you.

</blockquote>

<pre><code class="shell language-shell">$ mpicxx pi_reduce.cc -o pi_reduce; mpirun -np 4 --hostfile hosts pi_reduce 1000000000</code></pre>

<strong>Code Implementation</strong>: (<em>7 points</em>)

<ul>

 <li>Complete <code>pi_reduce.cc</code>: the MPI reduce communication for PI.</li>

 <li>Implement the code using <code>MPI_Reduce</code>.</li>

 <li>Do not modify the output messages.</li>

 <li><code>$ mpicxx pi_reduce.cc -o pi_reduce; mpirun -np 4 --hostfile hosts pi_reduce 1000000000</code> should run correctly.</li>

 <li>The argument of <code>-np</code> is expected to be any positive integer less than or equal to 16.</li>

 <li><code>mpirun -np 4 --hostfile hosts pi_reduce 1000000000</code> should run almost as fast as TA’s reference version. The difference (in execution time) should be within 15%.</li>

</ul>

<strong>Q6</strong> (<em>1 points</em>)

<ol>

 <li>Measure the performance (execution time) of the code for 2, 4, 8, 12, 16 MPI processes and plot it. (<em>1 points</em>)</li>

</ol>

<h4 id="126mpiwindowsandonesidedcommunicationlinearreductionalgorithm">1.2.6 MPI Windows and One-Sided Communication &amp; Linear Reduction Algorithm</h4>

Use MPI Windows and MPI one-sided communication, which we didn’t cover this in class. You can choose the functions you’d like to use but remember that there is a reduction on the same MPI window from many processes! You may refer to <code>one_side_example.c</code> to get familiar with it.

There is a starter code <code>pi_one_side.cc</code> for you.

<pre><code class="shell language-shell">$ mpicxx pi_one_side.cc -o pi_one_side; mpirun -np 4 --hostfile hosts pi_one_side 1000000000</code></pre>

<strong>Code Implementation</strong>: (<em>7 points</em>)

<ul>

 <li>Complete <code>pi_one_side.cc</code>: the MPI one sided communication for PI.</li>

 <li>Implement the code using <code>MPI_Widows_*</code> (or <code>MPI_Put</code>, <code>MPI_Get</code>, and <code>MPI_Accumulate</code>). You only need to implement the first one, and the remainings are optional.</li>

 <li>Do not modify the output messages.</li>

 <li><code>$ mpicxx pi_one_side.cc -o pi_one_side; mpirun -np 4 --hostfile hosts pi_one_side 1000000000</code> should run correctly.</li>

 <li>The argument of <code>-np</code> is expected to be any positive integer less than or equal to 16.</li>

 <li><code>mpirun -np 4 --hostfile hosts pi_one_side 1000000000</code> should run almost as fast as TA’s reference version. The difference (in execution time) should be within 15%.</li>

</ul>

<strong>Q7</strong> (<em>5 points</em>)

<ol>

 <li>Measure the performance (execution time) of the code for 2, 4, 8, 12, 16 MPI processes and plot it. (<em>1 points</em>)</li>

 <li>Which approach gives the best performance among the <strong><a href="#121--mpi-blocking-communication--linear-reduction-algorithm">1.2.1</a></strong>–<strong><a href="#126-mpi-windows-and-one-sided-communication--linear-reduction-algorithm">1.2.6</a></strong> cases? What is the reason for that? (<em>4 points</em>)</li>

</ol>

<h3 id="13measuringbandwidthandlatencyonnycuppworkstationswithpingpong">1.3 Measuring Bandwidth and Latency on NYCU-PP workstations with Ping-Pong</h3>

Expected knowledge is concepts of bandwidth and latency and performance model for parallel communication.

The ping-pong is a benchmark code to measure the bandwidth and latency of a supercomputer. In this benchmark, two MPI processes use <code>MPI_Send</code> and <code>MPI_Recv</code> to continually bounce messages off of each other until a final limit.

The ping-pong benchmark provides as output the average time for the ping-pong for different messages sizes.

If we plot the results of the ping-pong with size on the x-axis and ping-pong time on the y-axis, we will roughly obtain points distributed along a line. If we do a linear best fit of the obtained points, we will get the intercept and the slope of the line. The inverse of the line slope is the bandwidth while the intercept is the latency of the system.

We will use <code>ping_pong.c</code> to measure the ping-pong time for different message sizes.

<strong>Instructions</strong>. For completing this exercise, you may refer to this video: <a href="https://www.youtube.com/watch?v=1J1aURqnwt4" target="_blank" rel="noopener">Performance Modeling &amp; Ping-Pong</a>

Run the ping-pong code and calculate the bandwidth and latency for:

<ul>

 <li>case 1: intra-node communication (two processes on the same node), and</li>

 <li>case 2: inter-node communication (two processes on different nodes). You should consider more different nodes in this case.</li>

</ul>

<strong>Q8</strong> (<em>9 points</em>)

<ol>

 <li>Plot ping-pong time in function of the message size for cases 1 and 2, respectively. (<em>4 points</em>)</li>

 <li>Calculate the bandwidth and latency for cases 1 and 2, respectively. (<em>5 points</em>)</li>

</ol>

<h2 id="2part2matrixmultiplicationwithmpi">2. Part 2: Matrix Multiplication with MPI</h2>

Write a MPI program which reads an $n times m$ matrix $A$ and an $m times l$ matrix $B$, and prints their product, which is stored in an $n times l$ matrix $C$. An element of matrix $C$ is obtained by the following formula:

$$c<em>{ij} = sum</em>{k=1}^m a<em>{ik}b</em>{kj}$$

where $a<em>{ij}$, $b</em>{ij}$ and $c_{ij}$ are elements of $A$, $B$, and $C$, respectively.

<img decoding="async" alt="Matrix Figure" data-recalc-dims="1" data-src="https://i0.wp.com/user-images.githubusercontent.com/18013815/98935948-3f3b5900-251f-11eb-9568-5feb972a03e0.png?w=600&amp;ssl=1" class="lazyload" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==">

 <noscript>

  <img decoding="async" src="https://i0.wp.com/user-images.githubusercontent.com/18013815/98935948-3f3b5900-251f-11eb-9568-5feb972a03e0.png?w=600&amp;ssl=1" alt="Matrix Figure" data-recalc-dims="1">

 </noscript>

Your mission is to calculate the matrix multiplication as fast as possible. You may try many methods (Ex: tiling or some algorithms. Note that SIMD isn’t allowed here!) to achieve a higher performance. You are allowed to use any code on the Internet, but you need to re-write the code yourself, which means copying and pasting is not allowed. In addition, you are not allowed to use any third-party library; that is, you have to write your MPI program from scratch. Furthermore, each node is allowed to run a single-threaded process, so using OpenMP, Pthread, fork, GPU, etc. are not allowed.

<blockquote>

 If you are not sure about the rule, ask on our Facebook group! <strong>Input</strong>

</blockquote>

In the first line, three integers $n$, $m$ and $l$ are given separated by space characters.

In the following lines, the $n times m$ matrix $A$ and the $m times l$ matrix $B$ are given. All the numbers are integers.

<strong>Output</strong>

Print elements of the $n times l$ matrix $C$ ($c_{ij}$). Print a single space character after “each” element!

<strong>Constraints</strong>

<ol>

 <li>$1 ≤ n, m, l ≤ 10000$</li>

 <li>$0 ≤ a<em>{ij}, b</em>{ij} ≤ 100$</li>

</ol>

<strong>Sample Input</strong>

<pre><code>3 2 31 20 34 51 2 10 3 2</code></pre>

<strong>Sample Output</strong>

<pre><code>1 8 5 0 9 6 4 23 14 </code></pre>

Note: There is ” 
” at the end of each line!

<strong>Data Sets</strong>

There are two data sets. The table shows the credit, the range of $n$, $m$, and $l$, and the hosts used for each data set.

<table>

 <thead>

  <tr>

   <th>Set</th>

   <th>Credit</th>

   <th>n, m, l</th>

   <th>Hosts</th>

  </tr>

 </thead>

 <tbody>

  <tr>

   <td>1</td>

   <td><em>10 points</em></td>

   <td>300-500</td>

   <td>Any of four from <code>pp1</code>–<code>pp9</code></td>

  </tr>

  <tr>

   <td>2</td>

   <td><em>10 points</em></td>

   <td>1000-2000</td>

   <td><code>pp1</code>–<code>pp9</code></td>

  </tr>

 </tbody>

</table>

You may have different implementations for the two data sets.

Notice that only one process per node is allowed.

Here is an example hostfile:

<pre><code class="conf language-conf">pp1 slots=1pp3 slots=1pp5 slots=1pp7 slots=1</code></pre>

<strong>Q9</strong> (<em>5 points</em>)

<ol>

 <li>Describe what approach(es) were used in your MPI matrix multiplication for each data set.</li>

</ol>

<h2 id="3requirements">3. Requirements</h2>

<h3 id="31requirementsforpartone">3.1 Requirements for Part One</h3>

<ul>

 <li>Code implementation (following the instructions in Sections <a href="#121--mpi-blocking-communication--linear-reduction-algorithm">1.2.1</a>–<a href="#126-mpi-windows-and-one-sided-communication--linear-reduction-algorithm">1.2.6</a>

  <ul>

   <li><code>hello.cc</code></li>

   <li><code>pi_block_linear.cc</code></li>

   <li><code>pi_block_tree.cc</code></li>

   <li><code>pi_gather.cc</code></li>

   <li><code>pi_nonblock_linear.cc</code></li>

   <li><code>pi_one_side.cc</code></li>

   <li><code>pi_reduce.cc</code></li>

  </ul></li>

 <li>Answer the questions (marked with “<strong>Q1-Q8</strong>“) in a <strong>REPORT</strong> using <a href="https://hackmd.io/" target="_blank" rel="noopener">HackMD</a>. (Notice that in this assignment a higher standard will be applied when grading the quality of your report.)</li>

</ul>

<h3 id="32requirementsforparttwo">3.2 Requirements for Part Two</h3>

<ul>

 <li>Code implementation

  <ul>

   <li>You should write a program that fits the input/output criteria as described in Section <a href="#2-part-two-matrix-multiplication-with-mpi">2</a>.</li>

   <li>You should write a <code>Makefile</code> that aims to generate an executable called <code>matmul</code>. In addition, <code>main.cc</code>, which contains the <code>main()</code> function, must be a part of <code>matmul</code> and cannot be modified.</li>

   <li>Your program will be evaluated with command <code>$ make; mpirun -np %N% --hostfile %hosts% matmul &lt; %data_file%</code>, where <code>N</code> is 2-8, <code>hosts</code> is our hostfile, and <code>data_file</code> is the input.</li>

   <li>There are some data in the <code>/HW4/data</code> folder for testing. The data are available at <code>/HW4/data.zip</code> if you’d like to test them in your own machine(s). The testing script provided uses the data to verify your program. You don’t expect to pass only the testing data.</li>

  </ul></li>

 <li>Answer the questions (marked with “<strong>Q9</strong>“) in a <strong>REPORT</strong> using <a href="https://hackmd.io/" target="_blank" rel="noopener">HackMD</a>. (Notice that in this assignment a higher standard will be applied when grading the quality of your report.)</li>