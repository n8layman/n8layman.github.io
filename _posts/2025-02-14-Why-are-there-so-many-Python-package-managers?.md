---
layout: post
title: Why are there so many Python package managers?
subtitle: Neither the forest NOR the trees
share-img: /assets/img/standards.png
tags: [data, Python, why?]
---

I’m a data scientist that works primarily in R. Recently I’ve been on a
path to learn more about Python, which I initially installed via brew.
Unfortunately I almost immediately ran into the following error:

    error: externally-managed-environment

    × This environment is externally managed
    ╰─> To install Python packages system-wide, try apt install
        python3-xyz, where xyz is the package you are trying to
        install.

        If you wish to install a non-Debian-packaged Python package,
        create a virtual environment using python3 -m venv path/to/venv.
        Then use path/to/venv/bin/python and path/to/venv/bin/pip. Make
        sure you have python3-full installed.

        If you wish to install a non-Debian packaged Python application,
        it may be easiest to use pipx install xyz, which will manage a
        virtual environment for you. Make sure you have pipx installed.

        See /usr/share/doc/python3.11/README.venv for more information.

    note: If you believe this is a mistake, please contact your Python installation or OS distribution provider. You can override this, at the risk of breaking your Python installation or OS, by passing --break-system-packages.
    hint: See PEP 668 for the detailed specification.

Which led me down a rabbit hole of learning about Python virtual
environments. 

Virtual environments let you isolate project-specific
dependencies from your system installation of Python. The benefits are
1) if you accidentally screw something up it won't disrupt the system Python
environment and 2) your projects become more reproducible.

Eventually, what started as a simple attempt to set up a new project
and practice some Python left me feeling completely out to sea.
After days of wrestling with pip, poetry, conda, pipenv, and virtualenv,
I was left scratching my head about Python’s package management
landscape. I can’t help but wonder: what the hell happened?

Coming from R’s relatively straightforward package management ecosystem, with just {renv} handling most of my needs, I was overwhelmed by the
sheer number of Python package management tools. While choice is generally good, each one seemed to solve a
different piece of the puzzle, and every blog post or tutorial I read recommended a different approach. After spending what felt like weeks exploring
various options and dealing with environment conflicts, I eventually decided I just needed to make
a choice and pick something that would work well for my data science workflow.

I mean just look at the table I made summarizing my notes!

<table style="width:99%;">
<colgroup>
<col style="width: 12%" />
<col style="width: 29%" />
<col style="width: 57%" />
</colgroup>
<thead>
<tr class="header">
<th>Tool</th>
<th>Pros</th>
<th>Cons</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong><code>venv</code></strong></td>
<td><p>- Built into Python 3.3+</p>
<p>- Zero configuration</p></td>
<td><p>- Manual dependency management</p>
<p>- Python-only focus</p></td>
</tr>
<tr class="even">
<td><strong><code>conda</code></strong></td>
<td><p>- Cross-platform scientific stack</p>
<p>- Non-Python dependency support</p></td>
<td><p>- Large installation size</p>
<p>- Slower than alternatives</p></td>
</tr>
<tr class="odd">
<td><strong><code>micromamba</code></strong></td>
<td><p>- Lightweight Conda alternative</p>
<p>- Blazing-fast solver</p></td>
<td><p>- CLI-only experience</p>
<p>- Smaller community</p></td>
</tr>
<tr class="even">
<td><strong><code>poetry</code></strong></td>
<td><p>- All-in-one dependency/packaging</p>
<p>- Modern pyproject.toml workflow</p></td>
<td><p>- Complex resolver</p>
<p>- Limited conda integration</p>
<p>- History of forcing users towards unwanted version upgrades
|</p></td>
</tr>
<tr class="odd">
<td><strong><code>pipx</code></strong></td>
<td><p>- Isolated CLI tool management</p>
<p>- Prevents global pollution</p></td>
<td>- For installing python commands, not for installing project package
dependencies</td>
</tr>
<tr class="even">
<td><strong><code>PDM</code></strong></td>
<td><p>- PEP 582 pioneer</p>
<p>- Fast dependency resolution</p></td>
<td><p>- No system library support</p>
<p>- Smaller ecosystem</p></td>
</tr>
<tr class="odd">
<td><strong><code>pip-tools</code></strong></td>
<td><p>- Lightweight pip enhancement</p>
<p>- Deterministic locking</p></td>
<td><p>- Manual workflow</p>
<p>- Basic features</p></td>
</tr>
<tr class="even">
<td><strong><code>uv</code></strong></td>
<td><p>- Rust-powered speed demon</p>
<p>- Modern dependency resolver</p></td>
<td><p>- New/unproven</p>
<p>- Python-only</p></td>
</tr>
<tr class="odd">
<td><strong><code>hatch</code></strong></td>
<td><p>- Modern project builder</p>
<p>- Built-in version management</p></td>
<td><p>- Growing community</p>
<p>- Less mature plugin system</p></td>
</tr>
<tr class="even">
<td><strong><code>rye</code></strong></td>
<td><p>- Rust-based toolchain</p>
<p>- Integrated Python version control</p></td>
<td><p>- Experimental</p>
<p>- Rapidly changing API</p></td>
</tr>
<tr class="odd">
<td><strong><code>virtualenv</code></strong></td>
<td><p>- Python 2 compatibility</p>
<p>- Legacy standard</p></td>
<td><p>- Redundant with venv</p>
<p>- Requires separate install</p></td>
</tr>
<tr class="even">
<td><strong><code>setuptools</code></strong></td>
<td><p>- Traditional packaging</p>
<p>- Deep customization</p></td>
<td><p>- No lockfiles</p>
<p>- Complex setup.py</p></td>
</tr>
<tr class="odd">
<td><strong><code>flit</code></strong></td>
<td><p>- Simple pure-Python packaging</p>
<p>- Quick PyPI uploads</p></td>
<td>- Limited to basic projects</td>
</tr>
<tr class="even">
<td><strong><code>mamba</code></strong></td>
<td><p>- Conda-compatible speed upgrade</p>
<p>- Optimized C++ solver</p></td>
<td><p>- Conda dependency</p>
<p>- Less mature than conda</p></td>
</tr>
<tr class="odd">
<td><strong><code>pixi</code></strong></td>
<td><p>- Unified Python/R dependency management</p>
<p>- Built-in task automation</p>
<p>- Reproducible lockfiles</p></td>
<td><p>- Early-stage project but under active development</p>
<p>- Requires workflow adoption</p></td>
</tr>
<tr class="even">
<td><strong><code>pipenv</code></strong></td>
<td><p>- Combines pip + virtualenv</p>
<p>- Security-focused dependency locking</p></td>
<td><p>- Deprecated in PEP 665</p>
<p>- Performance issues</p>
<p>- Less active development</p></td>
</tr>
</tbody>
</table>

And I’m certain I missed more than a few.

After exploring these numerous options, I settled on
[pixi](https://github.com/prefix-dev/pixi) for my data science workflow,
and here’s why: I needed a tool that could
handle both Python packages and the scientific computing dependencies
that often come with data science work. What really sealed the deal was
discovering that pixi can handle R packages alongside Python ones - 
which could be a game-changer for hybrid workflows where I might 
need both languages.

While conda is the traditional choice here, pixi offers a more modern
and streamlined approach. Its `.toml` and `.lock` files ensure
reproducibility (similar to renv in R), while its ability to handle
non-Python dependencies means I don’t have to worry about complex
system-level installations or bother with docker for simple projects.

This is particularly valuable when working with packages that require
specific system libraries or external dependencies like database
drivers, CUDA toolkits, or specialized scientific computing libraries.

The `pixi.toml` configuration seems clean and intuitive and includes both
templates and activation scripts. Most importantly, the task automation
features let me define common data science workflows (like running
jupyter notebooks or training models) in a way that’s easily shareable
with colleagues.

While it’s true that pixi seems newer than some alternatives, it could
help me resolve a lot of my package management needs simply and easily.
And its unique ability to handle both R and Python ecosystems made it
the clear choice for my transition to a multi-language data science
workflow. I expect there will still be some friction but I gotta admit,
I’m excited. And I love the idea of being able to specify and install
the versions of languages, my most used packages, and any external
dependencies all from one place.  
  
I like the idea of being to open a shell and run something like:  
\`\`\`

    pixi add pytorch

or

    pixi add r-tidymodels

Have you got any suggestions? What do you use?
