# Introduction to Apptainer/Singularity
<!-- ALL-CONTRIBUTORS-BADGE:START - Do not remove or modify this section -->
[![All Contributors](https://img.shields.io/badge/all_contributors-4-orange.svg?style=flat-square)](#contributors-)
<!-- ALL-CONTRIBUTORS-BADGE:END -->

[![HSF Training Center][training-center-badge]][hsf-training-center]
[![Upcoming Events][schools-badge]][schools]
[![Twitter Follow][twitter-badge]][twitter]

[![pre-commit.ci status](https://results.pre-commit.ci/badge/github/hsf-training/hsf-training-singularity-webpage/gh-pages.svg)](https://results.pre-commit.ci/latest/github/hsf-training/hsf-training-singularity-webpage/gh-pages)
[![pages-build-deployment](https://github.com/hsf-training/hsf-training-singularity-webpage/actions/workflows/pages/pages-build-deployment/badge.svg)](https://github.com/hsf-training/hsf-training-singularity-webpage/actions/workflows/pages/pages-build-deployment)

This repository generates the corresponding lesson website from [The Carpentries](https://carpentries.org/) repertoire of lessons.

## What is Apptainer/Singularity?

Apptainer (formerly known as Singularity) is a free and open-source container platform that allows you to create and run applications in isolated images (also called "containers") in a simple, portable, fast, and secure manner. It performs operating system level virtualization known as containerization. Many container platforms are available, but Apptainer is designed to bring containers and reproducibility to the scientific community and High-Performance Computing (HPC) use cases. Using Apptainer, developers can work in reproducible environments of their choice and design, and these complete environments can be easily copied and executed on other platforms.

## 📅 Past events and videos

* 🎥 [Jan 2023](https://indico.cern.ch/event/1219810/)

Emoji key: 🎥 (full video recordings availabile), ⛏️ (hackathon)

## 🤗 Contributing
<!-- CENTRALLY MAINTAINED SECTION -->
<!-- Remove the above marker to disable having this section be overwritten -->

We welcome all contributions to improve the lesson! Maintainers will do their best to help you if you have any
questions, concerns, or experience any difficulties along the way.

If you make non-trivial changes (i.e., more than fixing a simple typo), you are eligible to be added to the [HSF Training Community page][hsf-training-community],
as well as to the list of contributors [below](#contributors-).

We'd like to ask you to familiarize yourself with our [Contribution Guide](CONTRIBUTING.md) and have a look at
the [more detailed guidelines][lesson-example] on proper formatting, ways to render the lesson locally, and even
how to write new episodes.

Quick summary of how to get a local preview: Install [jekyll][jekyll] and then run

```
bundle install
bundle update
bundle exec jekyll serve
```

Unless we change framework versions, only the last command needs to be typed after the first time.

Before committing anything, we also ask you to install the [pre-commit][pre-commit] hooks of this repository:

```bash
pip3 install pre-commit
pre-commit install
```

Please see the current list of [issues][issues] for ideas for contributing to this
repository. For making your contribution, we use the GitHub flow, which is
nicely explained in the chapter [Contributing to a Project][progit] in Pro Git
by Scott Chacon.
Look for the tag [![good_first_issue]][gfi-badge], which marks particularly simple issues to get you started.

<!-- END CENTRALLY MAINTAINED SECTION -->

## Citation

To cite this lesson, please consult with [CITATION](CITATION)

## 💖 Authors

Current maintainers of this lesson are

* Michel Villanueva ([michmx](https://github.com/michmx))

Thanks goes to these wonderful people ([emoji key](https://allcontributors.org/docs/en/emoji-key)):

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tbody>
    <tr>
      <td align="center"><a href="https://github.com/michmx"><img src="https://avatars.githubusercontent.com/u/2147367?v=4?s=100" width="100px;" alt="Michel H. Villanueva"/><br /><sub><b>Michel H. Villanueva</b></sub></a><br /><a href="#content-michmx" title="Content">🖋</a></td>
      <td align="center"><a href="https://github.com/mambelli"><img src="https://avatars.githubusercontent.com/u/1558058?v=4?s=100" width="100px;" alt="Marco Mambelli"/><br /><sub><b>Marco Mambelli</b></sub></a><br /><a href="#ideas-mambelli" title="Ideas, Planning, & Feedback">🤔</a> <a href="#content-mambelli" title="Content">🖋</a></td>
      <td align="center"><a href="https://github.com/amangoel185"><img src="https://avatars.githubusercontent.com/u/10528392?v=4?s=100" width="100px;" alt="Aman Goel"/><br /><sub><b>Aman Goel</b></sub></a><br /><a href="#content-amangoel185" title="Content">🖋</a></td>
      <td align="center"><a href="https://github.com/amanmdesai"><img src="https://avatars.githubusercontent.com/u/98302868?v=4?s=100" width="100px;" alt="Aman Desai"/><br /><sub><b>Aman Desai</b></sub></a><br /><a href="#content-amanmdesai" title="Content">🖋</a></td>
    </tr>
  </tbody>
</table>

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->

This project follows the [all-contributors](https://github.com/all-contributors/all-contributors) specification. Contributions of any kind welcome!



[lesson-example]: https://carpentries.github.io/lesson-example
[pre-commit]: https://pre-commit.com/
[hsf-training-community]: https://hepsoftwarefoundation.org/training/community
[hsf-training-center]: https://hepsoftwarefoundation.org/training/curriculum.html
[training-center-badge]: https://img.shields.io/badge/HSF%20Training%20Center-browse-ff69b4
[schools]: https://hepsoftwarefoundation.org/Schools/events.html
[issues]: https://github.com/hsf-training/hsf-training-singularity-webpage/issues
[progit]: http://git-scm.com/book/en/v2/GitHub-Contributing-to-a-Project
[jekyll]: https://jekyllrb.com/
[allcontrib-emoji-key]: https://allcontributors.org/docs/en/emoji-key
[gfi-badge]: https://img.shields.io/badge/-good%20first%20issue-gold.svg
[schools-badge]: https://img.shields.io/badge/upcoming%20events-browse-ff69b4
[twitter-badge]: https://img.shields.io/twitter/follow/hsftraining?style=social
[twitter]: https://twitter.com/hsftraining
