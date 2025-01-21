<!-- insert
---
title: Into Ledger
date: 2021-08-21T16:23:33
gometa: "joly.pw/into-ledger git https://github.com/cljoly/into-ledger"
description: "🔮 AI-powered expenses classifier for ledger. The original project seems not to be maintained anymore, hence this fork to continue improving it."
tags:
- Finance
- AI
- Ledger
---
end_insert -->
<!-- remove -->
<div align="center">

🔮 Into Ledger
==========
<!-- end_remove -->

<!-- insert
{{< github_badge >}}

{{< rawhtml >}}
<div class="badges">
{{< /rawhtml >}}
end_insert -->

[![Go Report Card](https://goreportcard.com/badge/github.com/cljoly/into-ledger)](https://goreportcard.com/report/github.com/cljoly/into-ledger) [![Test](https://github.com/cljoly/into-ledger/actions/workflows/build.yml/badge.svg)](https://github.com/cljoly/into-ledger/actions/workflows/build.yml) [![GitHub last commit](https://img.shields.io/github/last-commit/cljoly/into-ledger)](https://github.com/cljoly/into-ledger/commits/master)

<!-- insert
{{< rawhtml >}}
end_insert -->
</div>
<!-- insert
{{< /rawhtml >}}
end_insert -->

into-ledger helps categorization of CSV transactions and conversion into ledger format for consumption by [ledger-cli.org](http://ledger-cli.org/). It makes importing hundreds of transactions into ledger a breeze. I typically get close to a hundred transactions per account per month myself, which is why I wrote this tool.

> [!NOTE]
> This is a fork of [manishrjain/into-ledger](https://github.com/manishrjain/into-ledger)  (which appears to be unmaintained)

Features
------

- **Accurate**             : Uses a much more accurate tf-idf expense classifier than used by cantino/reckon.
- **Includes and Aliases** : Correctly parses your existing journal file, handling all includes and account aliases.
- **Keyboard Shortcuts**   : Assigns dynamic keyboard shortcuts, so classifying transactions is just a keystroke away.
- **Auto save**            : Uses temporary storage (boltdb) to persist transactions that you have categorized or acknowledged to be correctly categorized, so you can quit whenever you want, without the risk of losing the work done so far.
- **Deduplication**        : Deduplicates incoming transactions from CSV against the transactions already present in ledger journal. This allows an easy resume from a broken workflow.
- **Nice UI**              : Colors and formatting, because it's not just about getting things done. It's also about making them look nice!

This fork adds:
- **Fixes**: various improvement, in particular to the CSV parsing
- **Fuzzy selection**: with fzf
- **More documentation**: admittedly still in progress
- **More modern, cleaner codebase**: this fork [![Go Report Card](https://goreportcard.com/badge/github.com/leowzukw/into-ledger)](https://goreportcard.com/report/github.com/leowzukw/into-ledger) vs upstream  [![Go Report Card](https://goreportcard.com/badge/github.com/manishrjain/into-ledger)](https://goreportcard.com/report/github.com/manishrjain/into-ledger)
- **Support of payee aliases**: file with internal aliases *TODO*


This fork removes:
- **Plaid support**: with better CSV parsing, the tool is more generic and can use data from many more websites

Install
-------

### Binaries

If you understand the security implications, [grab a binary from the release page and run it](https://github.com/cljoly/into-ledger/releases/latest).

### Build it yourself

A better option, security-wise, is to install the go toolchain and run
```
go get -u joly.pw/into-ledger
```

Help
----
```
Usage of ./into-ledger:
  -a string
    	Name of bank account transactions belong to.
  -c string
    	Set currency if any.
  -conf string
    	Config directory to store various into-ledger configs in. (default "/home/mrjn/.into-ledger")
  -comma string
	Separator of fields in csv file
  -csv string
    	File path of CSV file containing new transactions.
  -d string
    	Express your date format in numeric form w.r.t. Jan 02, 2006, separated by slashes (/). See: https://golang.org/pkg/time/ (default "01/02/2006")
  -debug
    	Additional debug information if set.
  -ic string
    	Comma separated list of columns to ignore in CSV.
  -j string
    	Existing journal to learn from.
  -o string
    	Journal file to write to. (default "out.ldg")
  -opt string
	Extra option to pass to ledger commands
  -s int
    	Number of header lines in CSV to skip
```

Usage
-----

```
# Importing from Citibank Australia
$ into-ledger -j ~/ledger/journal.ldg -csv ~/ledger/ACCT_464_25_07_2016.csv --ic "3,4" -o out.data -a citi -c AUD -d "02/01/2006"

# Importing from Chase USA. Skips the first line in CSV file. Also skips the first (0) and second (1) column in csv. Outputs to out.data. Sets currency as USD.
$ into-ledger -j ~/ledger/journal.ldg -csv ~/ledger/Activity.CSV --ic "0,1" -o out.data -a chase -c USD -s 1
```

Having to specify these command line arguments over and over again is annoying. So, instead you can create a config file in "$HOME/.into-ledger/config.yaml", storing the flag values for reuse, like so:

```
accounts:
  chase:
    c: USD
    j: /home/mrjn/ledger/journal.ldg
    d: 01/02/2006
    ic: "0,1"
    o: /home/mrjn/ledger/chase.out
    s: 1
  cba-smart:
    c: AUD
    j: /home/mrjn/ledger/journal.ldg
    d: 02/01/2006
    ic: "3"
    o: /home/mrjn/ledger/cba.out
```

**Note: The way config is stored has changed recently. Please update your version of into-ledger using `go get -u -v github.com/manishrjain/into-ledger`. Also, update your config file.**

Now you can just run:
`into-ledger -a chase -csv <input-csv>`, or `into-ledger -a cba-smart -csv <input-csv>`

Dates
-----

into-ledger requires you to specify the date format in numeric form w.r.t. Jan 02, 2006. This is how Go language parses dates. This is frequently a cause of confusion among folks unfamiliar with the language. So, please find here the commonly used date formats and how to specify them in into-ledger.

| Formatting Style | Regions used in | into-ledger format |
| ---------------- | --------------- | ------------------ |
| Month/Day/Year | USA | 01/02/2006 |
| Month-Day-Year | USA | 01-02-2006 |
| Day/Month/Year | Australia, others | 02/01/2006 |
| Day-Month-Year | Australia, others | 02-01-2006 |
| Year/Month/Day | Ledger | 2006/01/02 |
| Year-Month-Day | Ledger | 2006-01-02 |


Keyboard Shortcuts
------------------

One of the great advantages of using `into-ledger` is how quickly you can categorize a transaction. Most of the times the underlying categorization algorithm is smart enough to do the right thing for you. However, for the rest, `into-ledger` shows you keyboard shortcuts to pick the right category.

`into-ledger` uses a keys module I wrote, which automatically assigns shortcuts to categories and persists them in `~/.into-ledger/shortcuts.yaml`. However, you might want to use certain keys for certain categories. In that case, feel free to hand-edit the `shortcuts.yaml` file. Just ensure that the same shortcut isn't being used twice in the file.

**Tip:** If you want to assign a shortcut to a category, but it's being used by another category, feel free to delete that category block from the shortcuts file. into-ledger will automatically reassign a new shortcut to the deleted category, and write it back.


Screenshots
-----------

**Parse transactions from CSV, and show automatically picked categories to be reviewed.**

![list of transactions](https://raw.githubusercontent.com/cljoly/into-ledger/master/list.png)

**Detect duplicates transactions in CSV, which are already present in ledger journal.**

![duplicate detection](https://raw.githubusercontent.com/cljoly/into-ledger/master/duplicates.png)

**Categorize transaction using persistent and dynamic keyboard shortcuts.**

![categorize transaction](https://raw.githubusercontent.com/cljoly/into-ledger/master/txn.png)
