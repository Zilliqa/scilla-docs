# Scilla Docs

Scilla short for Smart Contract Intermediate-Level LAnguage is a smart contract
language being developed for Zilliqa.

## Contributing

If you spot any issues or have any ideas on how we can improve the
documentation, help us log an issue
[here](https://github.com/Zilliqa/scilla-docs/issues)

## Installing dependencies

To compile the documentation you need the following dependencies.

1. You need to have [sphinx](http://www.sphinx-doc.org/en/master/) installed.
   Install Sphinx by running `pip install -U Sphinx`.
2. Our build system checks for spelling mistakes automatically (we use the UK
   spelling, by the way). To enable it some additional dependencies are needed.
   - The [sphinxcontrib.spelling][spelling] spelling cheker for Sphinx:
   `pip install -U sphinxcontrib-spelling`.
   - The command above will automatically install [PyEnchant][pyenchant] library
     which itself needs the C [Enchant][enchant] library which one has to
     install using e.g. your OS's package manager. You can find the installation
     instructions [here][enchant-install]. Basically, you want either
     `libenchant` or `enchant` package depending on your setup.

[spelling]: https://sphinxcontrib-spelling.readthedocs.io/en/latest/index.html
[pyenchant]: https://pyenchant.github.io/pyenchant/index.html
[enchant]: https://www.abisource.com/projects/enchant/
[enchant-install]: https://pyenchant.github.io/pyenchant/install.html#installing-the-enchant-c-library
   
## Building the docs
   
### Spellchecking

Run `make spell` from [docs](./docs/) folder. In case of any found spelling
mistakes you will see an output like the following.
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

