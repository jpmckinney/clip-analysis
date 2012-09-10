# Canadian Open Data Licenses

This repository complements the excellent work done by the [Samuelson-Glushko Canadian Internet Policy and Public Interest Clinic](http://cippic.ca/) in its [CIPPIC Open Licensing Project (CLIP)](http://clip.cippic.ca/) initiative.

## Dependencies

* Ruby 1.9.3 from [RVM](https://rvm.io/)
* [Homebrew](http://mxcl.github.com/homebrew/)
* [GNU Scientific Library (GSL)](http://www.gnu.org/software/gsl/)
* [Superfastmatch](https://github.com/mediastandardstrust/superfastmatch) (optional)

## Usage

### Preparation

We must download the licenses to perform any analysis on them:

    # From this repository's directory.
    bundle
    bundle exec rake download

### Analysis

#### Diff

    bundle exec rake similarity:diff

A difference algorithm returns the number of common characters between two texts. We use that number to calculate a [Sørensen-Dice coefficient](http://en.wikipedia.org/wiki/Dice%27s_coefficient), the similarity measure for this analysis.

We use [Neil Fraser](http://neil.fraser.name/)'s [Diff-Match-Patch](http://code.google.com/p/google-diff-match-patch/), which implements [Myer's difference algorithm](http://neil.fraser.name/software/diff_match_patch/myers.pdf).

This difference algorithm is sensitive to word order changes and paraphrasing. Thankfully, the order of clauses in licenses is somewhat consistent, and most jurisdictions copy-paste license text, unless they are authoring a new license.

#### Term Frequency–Inverse Document Frequency

    bundle exec rake similarity:tfidf

Uses a [document-term matrix](http://en.wikipedia.org/wiki/Document-term_matrix) in which the values are tf*idf weights to calculate similarities. Uses our own [tf-idf-similarity](http://github.com/opennorth/tf-idf-similarity) gem.

#### Superfastmatch

Start Superfastmatch and load the licenses into it:

    # From the superfastmatch/ directory.
    make run
    scripts/load.sh path/to/data/directory

### Visualize

View a similarity matrix from the command-line, for example:

    bundle exec rake print[similarity-diff.yml]

## See Also

* [Gensim](http://radimrehurek.com/gensim/) (Python)

## Bugs? Questions?

The main repository is on GitHub: [http://github.com/opennorth/canadian-open-data-licenses](http://github.com/opennorth/canadian-open-data-licenses), where your contributions, forks, bug reports, feature requests, and feedback are greatly welcomed.

Copyright (c) 2012 Open North Inc., released under the MIT license
