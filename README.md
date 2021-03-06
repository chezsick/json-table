# json-table [![Build Status](https://travis-ci.org/micha/json-table.svg?branch=master)](https://travis-ci.org/micha/json-table)

**Jt** reads UTF-8 encoded JSON forms from `stdin` and writes tab separated
values (or CSV) to `stdout`. A simple stack-based programming language is used
to extract values from the JSON input for printing.

## OVERVIEW

Extracting information from deeply nested JSON data is difficult and unreliable
with tools like **sed** and **awk**, and tools that are specially designed for
manipulating JSON are cumbersome to use in the shell because they either return
their results as JSON or introduce a new turing complete scripting language
that needs to be quoted and constructed via string interpolation.

**Jt** provides only what is needed to extract data from nested JSON data
structures and organize the data into a table. Tools like **cut**, **paste**,
**join**, **sort**, **uniq**, etc. can be used to efficiently reduce the
tabular data to produce the final result.

#### Features

* **Self contained** &mdash; statically linked, has no build or runtime dependencies.
* **Fast, small memory footprint** &mdash; efficiently process large JSON input.
* **Correct** &mdash; parser does not accept invalid JSON (see tests for details).

#### Example 1

Suppose we have some JSON data in a log file that we want to process:

```json
{"account":123,"amount":1.00}
{"account":789,"amount":2.00}
{"account":123,"amount":3.00}
{"account":123,"amount":4.00}
{"account":456,"amount":5.00}
```

First, use **jt** to extract interesting values to get us out of JSON-world and
into a nice tab delimited, newline separated tabular format that is amenable to
processing with shell utilities:

```bash
jt [ account % ] amount % <<EOT
{"account":123,"amount":1.00}
{"account":789,"amount":2.00}
{"account":123,"amount":3.00}
{"account":123,"amount":4.00}
{"account":456,"amount":5.00}
EOT
```
```
123     1.00
789     2.00
123     3.00
123     4.00
456     5.00
```

From here we can process the values in the shell. For example, to compute the
sum of the amounts for account 123:

```bash
jt [ account % ] amount % <<EOT | awk -F\\t '$1 == 123 {print $2}' | paste -sd+ |bc
{"account":123,"amount":1.00}
{"account":789,"amount":2.00}
{"account":123,"amount":4.00}
{"account":123,"amount":4.00}
{"account":456,"amount":5.00}
EOT
```
```
9.00
```

Or to compute the amount frequencies for the account:

```bash
jt [ account % ] amount % <<EOT | awk -F\\t '$1 == 123 {print $2}' | sort | uniq -c
{"account":123,"amount":1.00}
{"account":789,"amount":2.00}
{"account":123,"amount":4.00}
{"account":123,"amount":4.00}
{"account":456,"amount":5.00}
EOT
```
```
      1 1.00
      2 4.00
```

#### Example 2

**Jt** can also extract data from nested JSON:

```bash
jt asgs [ name % ] instances [ id % ] [ az % ] [ state % ] <<EOT
{
  "asgs": [
    {
      "name": "test1",
      "instances": [
        {"id": "i-9fb75dc", "az": "us-east-1a", "state": "InService"},
        {"id": "i-95393ba", "az": "us-east-1a", "state": "Terminating:Wait"},
        {"id": "i-241fd0b", "az": "us-east-1b", "state": "InService"}
      ]
    },
    {
      "name": "test2",
      "instances": [
        {"id": "i-4bbab16", "az": "us-east-1a", "state": "InService"},
        {"id": "i-417c312", "az": "us-east-1b", "state": "InService"}
      ]
    }
  ]
}
EOT
```
```
test1   i-9fb75dc       us-east-1a      InService
test1   i-95393ba       us-east-1a      Terminating:Wait
test1   i-241fd0b       us-east-1b      InService
test2   i-4bbab16       us-east-1a      InService
test2   i-417c312       us-east-1b      InService
```

The resulting TSV data can be piped to **awk**, for example, to get just the
instances in `test1` that are in service:

```bash
jt asgs [ name % ] instances [ id % ] [ az % ] [ state % ] <<EOT \
  | awk -F\\t '$1 == "test1" && $4 == "InService" {print}'
{
  "asgs": [
    {
      "name": "test1",
      "instances": [
        {"id": "i-9fb75dc", "az": "us-east-1a", "state": "InService"},
        {"id": "i-95393ba", "az": "us-east-1a", "state": "Terminating:Wait"},
        {"id": "i-241fd0b", "az": "us-east-1b", "state": "InService"}
      ]
    },
    {
      "name": "test2",
      "instances": [
        {"id": "i-4bbab16", "az": "us-east-1a", "state": "InService"},
        {"id": "i-417c312", "az": "us-east-1b", "state": "InService"}
      ]
    }
  ]
}
EOT
```
```
test1   i-9fb75dc       us-east-1a      InService
test1   i-241fd0b       us-east-1b      InService
```

## INSTALL

Linux users can install prebuilt binaries from the release tarball:

```
sudo bash -c "cd /usr/local && wget -O - https://github.com/micha/json-table/raw/master/jt.tar.gz | tar xzvf -"
```

Otherwise, build from source:

```
make && make test && sudo make install
```

> **NOTE:** Previous versions installed the **jt** manual in the `$PREFIX/man/`
> directory, which was incorrect. They are now installed into `$PREFIX/share/man/`.
> If you have installed **jt** previously you will probably want to delete those
> old man pages from the `$PREFIX/man/` directory if you install a newer version.

## DOCUMENTATION

See the [man page][man] or `man jt` in your terminal.

## EXAMPLES

The [man page][man] has many examples.

## SEE ALSO

**Jt** is based on ideas from the excellent [jshon][jshon] tool.

## COPYRIGHT

Copyright © 2017 Micha Niskin. Distributed under the Eclipse Public License.

[man]: http://htmlpreview.github.io/?https://raw.githubusercontent.com/micha/json-table/master/jt.1.html
[jshon]: http://kmkeen.com/jshon/
