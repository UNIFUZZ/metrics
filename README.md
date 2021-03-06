# UNIFUZZ Metrics
Collecting and analyzing various metrics from fuzzing experiments.

The code here is **NOT meant to be directly used** in your fuzzing result analysis, but to give some demonstrations and provide some insights.

## Prerequisite

1. Running the crash file to obtain the output of ASAN and GDB.

RUN with ASAN: [crashrunner.py](./code/crashrunner.py)

RUN with GDB Exploitable: [exploitablerunner.py](./code/exploitablerunner.py)

**Insights:** Before executing the crash file, copy the file to another place, in case some program may override the input file (and thus destroy the file).

2. Load the output to database

[db_crash_init.py](./code/db_crash_init.py)

The database schema is provided here: [unifuzz.sql](./code/unifuzz.sql)

**Insights:** Using database to mange your experiment data.

3. Load the docker containers start time to database `dockers` table

[containers_starttime.py](./code/containers_starttime.py)

**Insights:** The precise timestamp of starting the experiment (docker container) can be obtained using the modification time of `hostname` file. (Timezone independent)

## Quantity of Unique Bugs

We suggest using top 3 functions from ASAN output to de-duplicate bugs.

And we also use GDB to complement bugs like FloatingPointException.

Then CVE matching is conducted, comparing full stacktrace with existing CVEs to give possible matches. Final matching results are determined by human judgment.

This step requires running all crash samples using ASAN and GDB (with Exploitable), and will record `bugid, Exploitable class, CVE id` for each file if the crash verified.

Each fuzzing repetition should get its count of found bugs, so we can generate boxplot Figure **unique bugs detected by fuzzers** and Table **p value and A12 score of unique bugs** for comparing different fuzzers.

Related SQL query:

```
select fuzzer, progname, dupN, count(distinct bugid) from crash where bugid>0 group by fuzzer, progname, dupN
```

Plot code: [plot_boxplot_bug_realword_all.py](./code/plot_boxplot_bug_realword_all.py)

## Quality of Bugs

### Severity

By matching a bug to CVE, we can use CVSS score as a metric to imply bug severity.

CVE Matching database file can be found [here: unibench_cve.sql](https://github.com/unifuzz/unibench_cvefeed/raw/master/unibench_cve.sql), Note: this database is manually constructed and may contain errors.

Besides, GDB plugin Exploitable also can give a report of whether this bug is exploitable.

This step depends on CVE matching and Exploitable report, and generates Table **CVEs with high severity** and **unique EXPLOITABLE bugs**.

### Rarity

It is intuitive that a bug that can be found by fewer fuzzers is relatively harder to be found.

We treat a bug that can only found by one fuzzer as a rare bug.

This metric is from comparing different fuzzers among an evaluation campaign, and generates Table **unique rare bugs**.

## Speed of Finding Bugs

To use this metric, create time of each crash file should be recorded (relative to the start of the experiment).

This metric generates Figure **unique bugs found over time**.

## Stability of Finding Bugs

RSD among 30 repetitons.

## Coverage

We use afl-cov to measure line coverage for each repetition. 

Boxplot **line coverage** and Figure **Spearman's correlation coefficient between unique bugs and line coverage** are generated for this metric.

## Overhead

During the fuzzing process, we collect `docker stats` output every ten minutes, this can give us the CPU and memory consumption information.

This step generates Figure **memory consumption of each fuzzer**.

