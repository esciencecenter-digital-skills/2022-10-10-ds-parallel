![](https://i.imgur.com/iywjz8s.png)


# Collaborative Document Parallel Python Day 2

2022-10-10 Parallel Python.

Welcome to The Workshop Collaborative Document.

This Document is synchronized as you type, so that everyone viewing this page sees the same text. This allows you to collaborate seamlessly on documents.


## 👮Code of Conduct

Participants are expected to follow these guidelines:
* Use welcoming and inclusive language.
* Be respectful of different viewpoints and experiences.
* Gracefully accept constructive criticism.
* Focus on what is best for the community.
* Show courtesy and respect towards other community members.

## ⚖️ License

All content is publicly available under the Creative Commons Attribution License: [creativecommons.org/licenses/by/4.0/](https://creativecommons.org/licenses/by/4.0/).

## 🙋Getting help

To ask a question, just raise your hand.

If you need help from a helper, place a pink post-it note on your laptop lid. A helper will come to assist you as soon as possible.

## 🖥 Workshop website

[link](https://esciencecenter-digital-skills.github.io/2022-10-10-ds-parallel/)

🛠 Setup

[link](https://esciencecenter-digital-skills.github.io/2022-10-10-ds-parallel/#software-setup)

Download files

[link](https://github.com/esciencecenter-digital-skills/parallel-python-workshop.git)


## 🗓️ Agenda
| Time | Topic |
|--:|:---|
|09:30|	Welcome and recap|
|09:45|	Delayed evaluation with Dask|
|10:30|	Break|
|10:40|	Parallel design patterns with Dask Bags|
|11:30|	Break|
|11:40|	Dependency based programming with Snakemake|
|12:30|	Lunch Break|
|13:30|	Big exercise in subgroups|
|15:30|	Break|
|15:40|	5-min presentations + discussion|
|16:15|	Post-workshop Survey|
|16:30|	Drinks|


## 🏢 Location logistics
* Coffee and toilets are in the hallway, just outside of the classroom.
* If you leave the building,
  be sure to be accompanied by someone from the eScience Center to let you back in through the ground floor door
* For access to this floor you might need to ring the doorbell so someone can let you in
* In case of an emergency, you can exit our floor using the main staircase.
  Or follow green light signs at the ceiling to the emergency staircase.
* **Wifi**: Eduroam should work. Otherwise use the 'matrixbuilding' network, password should be printed out and available somewhere in the room.

## 🔧 Exercises

### Exercise: run the workflow
Given this workflow:
~~~python
x_p = add(1, 2)
y_p = add(x_p, 3)
z_p = add(x_p, -3)
~~~
Visualize and compute `y_p` and `z_p`, how often is `x_p` evaluated?


Now change the workflow:
~~~python
x_p = add(1, 2)
y_p = add(x_p, 3)
z_p = add(x_p, y_p)
~~~
We pass the yet uncomputed promise `x_p` to both `y_p` and `z_p`. How often do you expect `x_p` to be evaluated? Run the workflow to check your answer.

Hint: try visualizing the workflow
~~~python
z_p.visualize(rankdir="LR")
~~~


### Exercise: create a dependency diagram
In a new directory create the following `Snakefile` (but it also exists already in the `snakemake_intro` folder of your download).

~~~python
rule all:
    input:
        "allcaps.txt"

rule generate_message:
    output:
        "message.txt"
    shell:
        "echo 'Hello, World!' > {output}"

rule shout_message:
    input:
        "message.txt"
    output:
        "allcaps.txt"
    shell:
        "tr [a-z] [A-Z] < {input} > {output}"
~~~

Alternatively, the same can be done using only Python commands:

~~~python
rule all:
    input:
        "allcaps.txt"

rule generate_message:
    output:
        "message.txt"
    run:
        with open(output[0], "w") as f:
            print("Hello, World!", file=f)

rule shout_message:
    input:
        "message.txt"
    output:
        "allcaps.txt"
    run:
        with open(output[0], "w") as f_out, open(input[0], "r") as f_in:
            content = f_in.read()
            f_out.write(content.upper())
~~~

1. View the dependency diagram: `snakemake --dag | dot | display` (or `snakemake --dag | dot -Tsvg > dag.svg`, to create a file), and run the workflow `snakemake -j1`
What is happening?

2. Create a new rule that concatenates `message.txt` and `allcaps.txt` (use the `cat` command).

3. Change the `all` rule to require the new output file. When you rerun the workflow, are all the steps repeated?

4. If we remove `allcaps.txt`, and rerun the workflow (e.g. with `snakemake -c 4`). What steps will now be run?



## 🧠 Collaborative Notes

### Recap by Leon

* How to measure performance and memory use
* Parallellisation using dask.array
* Calculating pi
* Enhancing Python performance using Numba
* Multithreading and lifting the Global Interpreter Lock (GIL)
* Multiprocessing
* How to access the result of the computation of a thread: use a queue:
```python=
import multiprocessing as mp
from threading import Thread

queue = mp.Queue()

def mysum(values, q):
    result = sum(values)
    q.put(result)

t2 = Thread(target=mysum, args=([1,2,3,4], queue))

t2.start()
t2.join()

queue.get()
```


### Dask delayed
- Code oriented tool, independent from Dask array
- Resembles numba.jit, as it is also a decorator, but works differently
- Works with Promises

Q: Why do we need promises, I just want the result!
A: If you have a promise, you can chain things together


Let's start from a new notebook!

```python=
from dask import delayed
```

```python=
@delayed
def add(a, b):
    result = a + b
    print(f"{a} + {b} = {result}")
    return a + b
```

```python=
x_p = add(1, 2)
```

```python=
type(x_p)
```
shows that this is not an actual result yet, but a promise!

Then to compute the result:
```python=
x_p.compute()
```

Let's make a workflow more complex and interdependent:
```python=
y_p = add(x_p, 3)
z_p = add(x_p, y_p)
```

Dask delayed lets you visualize your workflow!
```python=
z_p.visualize(rankdir = "LR")
```
(The rankdir argument is to change the automatic visualization that goes top to bottom.)


_(exercise moved to [the Exercise header](https://hackmd.io/UemLppouTGWjEWhbGMc9KA?both#%F0%9F%94%A7-Exercises).)_





Recall:
```python=
%%timeit

if __name__ == "__main__":
    N = 10**7
    rnd1 = np.random.random(N)
    rnd2 = np.random.random(N)
    rnd3 = np.random.random(N)

    p1 = Process(target=np.sort, args = (rnd1,))
    p2 = Process(target=np.sort, args = (rnd2,))
    p3 = Process(target=np.sort, args = (rnd3,))

    p1.start()
    p2.start()
    p3.start()

    p1.join()
    p2.join()
    p3.join()
```

### Groundwork
```python=
@delayed
def add(*args):
    return sum(args)
```

`sum(1,2,3,4)` gives an error; `add(1,2,3,4` works!


In preparing towards averaging the result of delayed functions, I want to have a function that can gather the result:
```python=
@delayed
def gather(*args):
    return list(args)
```

The input of this will be a list of promises, the output is the promise of a list 🙃

Q: How is this different from append?
A: Then you would still have a list of promises. We need a single promise.

```python=
x_p = gather(*(add(n, n) for n in range(10)))
x_p.visualize()
```

Tip: if you visualize a delayed workflow, aspects that are not delayed will not show up in this workflow.

NB: The `*()` in the argument turns every add function into an individual argument. It is similar to a list comprehension, except that it is not a single object (namely a list), but individual objects.

```python=
x_p.compute()
```
To get the result; which is a list of results of `add()`.

#### Please write a delayed function that computes the mean of more than two numbers.
```python=
@delayed
def mean(*args):
    return(sum(args)/len(args))
```
Verify that the function is correct by performing a `.compute()`

```python=
m_p = mean(1,2,3,4)
m_p.compute()
```

Comment: another argument for `.visualize()` is `color="order"`, which colors the operations in the order that they are executed (and creates a very festive figure).

#### Define the calc_pi function from yesterday
```python=
import random
def calc_pi(N):
    M = 0
    for i in range(N):
        x = random.uniform(-1, 1)
        y = random.uniform(-1, 1)
        dist = x**2 + y**2
        if dist < 1:
            M += 1
    return 4 * M / N
```

```python=
%timeit calc_pi(10**6)
```

#### Now we can run calc_pi in parallel using dask delayed
```python=
N = 10**6
# mean here is the delayed mean function you wrote yourself
pi_p = mean(*(delayed(calc_pi)(N) for _ in range(10)))
pi_p.compute()
```

```python=
%timeit pi_p.compute()
```

We are running the calc_pi function ten times in parallel. Ideally this would take as long as a single run, but it doesn't!
There are two reasons:
- your cpu probably has fewer than 10 cores, so not all 10 can run in parallel
- dask delayed uses threads for parallelization. This means that any function that does not circumvent the GIL actually cannot run in parallel.

We fill fix the second point with the numba nogil version of calc_pi, that we also developed yesterday.

```python=
import numba
@numba.jit(nogil=True)
def calc_pi_numba(N):
    M = 0
    for i in range(N):
        x = random.uniform(-1, 1)
        y = random.uniform(-1, 1)
        dist = x**2 + y**2
        if dist < 1:
            M += 1
    return 4 * M / N
```

Run it at least once to compile it
```python=
%timeit calc_pi_numba(10**6)
```

Now run it several times in parallel. E.g. 4 times if you have 4 cores: all 4 should then run in parallel.
```python=
N = 10**6
# mean here is the delayed mean function you wrote yourself
pi_p = mean(*(delayed(calc_pi_numba)(N) for _ in range(10)))
pi_p.compute()
```

It does speed up, partly from the numba compiler, partly from the parallel computations.

### Dask bags
bags = bags of data 🙃

```python=
import dask.bag as db

bag = db.from_sequence(["mary", "had", "a", "little", "lamb"])


```
You can use bags for many different operations. It opens a toolbox, including:
|tool | # elements in bag | # elements in output |
|:---|:--:|:--:|
|`map`|n|n|
|`filter`|n|<n|
|`groupby`|n|<n|
|`reduction`|n|1|



```python=
def f(x):
    return x.upper()

bag.map(f).compute()
bag.map(f).visualize()
```

```python=
def pred(x):
    return "a" in x

bag.filter(pred).compute()
```

Exercise: replace `filter` in the previous code by `map`. Before you run it: what do you think the output will be?

A: It returns a list of `True` and `False`, with the same length of the bag.

`reduction` does an aggregation on two levels (i.e. two aggregations, one on top of the other).
```python=
def count_chars(x):
    per_word = [len(w) for w in x]
    return sum(per_word)

bag.reduction(count_chars, sum).visualize()
bag.reduction(count_chars, sum).compute()
```

The eventual purpose of our current project is to know how many unique word-stems are in a book.

The algorithms have been developed by Martin Porter. These stemmers are called Snowball, because Porter created a programming language with this name for creating new stemming algorithms. There is more information available at http://snowball.tartarus.org/

There is a library that can find word stems. We also need requests, to download books.
```python=
from nltk.stem.snowball import PorterStemmer
import requests
stemmer = PorterStemmer()

def good_word(w):
    return len(w) > 0 and not any(i.isdigit() for i in w)

def clean_word(w):
    #return w.strip("*!?.:;'\",“’‘”()_").lower()
    return ''.join([i for i in s if i.isalpha()])

def load_url(url):
    response = requests.get(url)
    return response.text
```

The books are:

- Mapes Dodge - https://www.gutenberg.org/files/764/764-0.txt
- Melville - https://www.gutenberg.org/files/15/15-0.txt
- Conan Doyle - https://www.gutenberg.org/files/1661/1661-0.txt
- Shelley - https://www.gutenberg.org/files/84/84-0.txt
- Stoker - https://www.gutenberg.org/files/345/345-0.txt
- E. Bronte - https://www.gutenberg.org/files/768/768-0.txt
- Austen - https://www.gutenberg.org/files/1342/1342-0.txt
- Carroll - https://www.gutenberg.org/files/11/11-0.txt
- Christie - https://www.gutenberg.org/files/61262/61262-0.txt

Or, as a python list:
```python!
all_books = ["https://www.gutenberg.org/files/764/764-0.txt", "https://www.gutenberg.org/files/15/15-0.txt", "https://www.gutenberg.org/files/1661/1661-0.txt", "https://www.gutenberg.org/files/84/84-0.txt", "https://www.gutenberg.org/files/345/345-0.txt", "https://www.gutenberg.org/files/768/768-0.txt", "https://www.gutenberg.org/files/1342/1342-0.txt", "https://www.gutenberg.org/files/11/11-0.txt", "https://www.gutenberg.org/files/61262/61262-0.txt"]
```

```python=
my_url = all_books[0]
my_book = load_url(my_url)

my_bag = db.from_sequence(my_book.split())
```

Let's apply some of the methods we learned:
```python!
my_bag.map(clean_word).filter(good_word).map(stemmer.stem).distinct().count().compute
```
`distinct` is another tool in dask, that finds unique occurrences


### Snakemake
Snakemake is a tool that helps write workflows, and it can parallelize inside this workflow.

- it's smart, it can identify steps that have been run before and not run it again
- meant to solve the problem of having to run multiple steps, and defining their order explicitly

(Check the folder `snakemake/` in the download!)

Create a new folder, e.g. `snakemake_peasoup/`, then a new file named `Snakefile` (no extension).

We will write a snakefile for pea soup:
```
rule all:
    input:
        "pea-soup.txt"

```
We can run this file in the terminal:
```bash
cd snakemake_peasoup
snakemake -c 1
```
(Be sure to run `snakemake` in the directory that contains `Snakefile`.)

The workflow fails, as `pea-soup.txt` does not exist. We need to create it, with a new rule:
```
rule pea_soup:
    input:
        "protosoup.txt"
    output:
        "pea-soup.txt"
    run:
        boil("protosoup.txt", "pea-soup.txt")
```
where `boil()` is a python function.

We can simplify this, by reusing the input and output elements of the rule, like so:
```
rule pea_soup:
    input:
        "protosoup.txt"
    output:
        "pea-soup.txt"
    run:
        boil(input[0], output[0])
```
The more complete the snakefile is, the better it can determine which things can run in parallel.

A rule can have no input! (e.g. the first rule that is run)

The workflow now fails again, as `protosoup.txt` also does not exist.

[The complete peasoup snakefile is here](https://github.com/esciencecenter-digital-skills/parallel-python-workshop/blob/main/peasoup/Snakefile). (Also in the download folder, in `peasoup/Snakefile`)

We can create the dependency diagram in the terminal (this might not work for everyone):
```bash
snakemake --dag | dot -Tsvg > dag.svg
```
(where `snakemake --dag` is the command, and the rest is to turn it into an image).

Run it with 4 cores:
```bash
snakemake -c 4
```
Running it a second time, shows the power of snakemake: this takes no time, as everything was done already, and intermediate files that were created in the previous run now already exist. It skips those steps to save time.

If files get updated in the meantime, snakemake can identify this (e.g. if an intermediate file is newer than its resulting output), and recreate the output after this intermediate file.

Running
```bash
snakemake --dry-run
```
shows an explanation of what will be done, without actually running it.



## 📚 Resources

### 📝 Profiling
#### Install line profiler
`conda install -y -c anaconda line_profiler`

#### Run in Notebook
```
%load_ext line_profiler
%lprun -f compute_mandelbrot compute_mandelbrot()
```

#### Output
```
Timer unit: 1e-06 s

Total time: 5.91596 s
File: /tmp/ipykernel_73246/1936148662.py
Function: compute_mandelbrot at line 15

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
    15                                           def compute_mandelbrot(width=WIDTH, height=HEIGHT, center=CENTER, extent=EXTENT, max_iter=MAX_ITER):
    16         1       4720.0   4720.0      0.1      niters = np.zeros((width, height), int)
    17         1          9.0      9.0      0.0      scale = max(extent.real / width, extent.imag / height)
    18
    19                                               # Loop through all selected complex number within the extent
    20       257        125.0      0.5      0.0      for h in range(height):
    21     65792      33046.0      0.5      0.6          for w in range(width):
    22                                                       # calculate the complex value c corresponding to the pixel position given by (w, h)
    23                                                       # // is floor division: the result is an integer if the inputs are integer
    24     65536      64297.0      1.0      1.1              c = center + (w - width // 2 + (h - height // 2) * 1j) * scale
    25     65536    5758903.0     87.9     97.3              k = mandelbrot(c, max_iter)
    26     65536      54859.0      0.8      0.9              niters[h, w] = k
    27         1          0.0      0.0      0.0      return niters
```