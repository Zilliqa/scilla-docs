# Scilla Docs

![](https://github.com/Zilliqa/scilla-docs/workflows/build/badge.svg)

Scilla short for Smart Contract Intermediate-Level LAnguage is a smart contract
language being developed for Zilliqa.

## Contributing

If you spot any issues or have any ideas on how we can improve the
documentation, help us log an issue
[here](https://github.com/Zilliqa/scilla-docs/issues).

## Installing dependencies

To compile the documentation you need the following system packages:
[Python3](https://www.python.org/downloads/) with the
[pip](https://pypi.org/project/pip/) package and
[libenchant](https://www.abisource.com/projects/enchant/) used to check
spelling.

On Debian-based systems you can install them with:
```
sudo apt-get install python3 python3-pip libenchant-2-dev
```

Then clone this repository and install the required Python packages from
[`requirements.txt`](https://github.com/Zilliqa/scilla-docs/blob/master/requirements.txt):
```
pip3 -U install -r requirements.txt
```

## Building the docs

### Spellchecking

Run `make spell` from [docs](./docs/) folder. In case of any found spelling
mistakes you will see an output like the following:
```
Spelling checker messages written to /path/to/scilla-docs/docs/build/spelling/output.txt
WARNING: Found 1 misspelled words
```
Checkout the `output.txt` file and fix the typos.

If you need to teach the spelling checker more words, add them to the
[spelling_wordlist.txt](./docs/source/spelling_wordlist.txt) file. The format is
really simple -- it's just one word per line the file. And the file is sorted in
ascending order.

### Building HTML docs

Run `make html` from [docs](./docs/) folder and make sure that the edits are
rendered correctly on the HTML files. To do that point your browser at the
locally built `/path/to/scilla-docs/docs/build/html/index.html` file and start
checking from it.

### Before submitting a pull request

Please preview your HTML files _before_ submitting a pull request. Try to
[squash](https://blog.github.com/2016-04-01-squash-your-commits/) your commits
before making the pull request. We know it's difficult, but it helps us keep our
commit logs clean and makes the reviewers' lives easier.

