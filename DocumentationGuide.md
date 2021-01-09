# ** A Whirlwind Tour of RHEOS Documentation **

This document is a short overview of how the RHEOS documentation system works.

A high level overview of the documentation is as follows. The documentation source files are mixture of pure markdown files (for regular documentation pages) and Julia files (that will be usable as both regular documentation and also Jupyter notebooks). The documentation structure is specified in the file `make.jl` using a dictionary data structure. The documentation can then be built locally by the user, and is built automatically on `git push` by the the GitHub action defined in `RHEOS/.github/.workflows/Docbuild.yml`.
<br><br>

# Local Documentation

## Directory Structure
<br>

### *User Modified Files*

Almost all the documentation related files are in the `RHEOS/docs` directory. The file within this directory that determines the documentation build and deployment process is in this directory and is called `make.jl`. It contains all the various documentation build parameters as well as the page structure of the documentation - discussed further below. There are two conceptual parts of `make.jl`: the first makes use of `Literate.jl` to build all the Jupyter notebooks and markdown files from the Julia source documentation files and put them in `RHEOS/docs/staging-docs`; the second uses `Documenter.jl` to convert the markdown files into HTML, and then deploys these if building remotely, if building locally deployment is skipped.

The rest of the files that are user-modified live in the `RHEOS/docs/src` folder. This is where there all of the markdown files are placed. It also contains another subfolder `RHEOS/docs/src/assets` where non-markdown files live such as the RHEOS logo, plots and other images that might need to be displayed in the documentation.

The one exception to the above is the source file for the 'home' page of the documentation. To keep everything synchronised, the `README.md` file is copied by `make.jl` into `RHEOS/docs/staging-docs/index.md`, with the RHEOS logo stripped away as it is already shown in the sidebar.
<br><br>

### *The Doc Environment, Separate from the Package Environment*

So packages that are only used in documentation do not have to be included in RHEOS' dependencies, we follow current best practice which is to have a separate environment (`Project.toml`) defined in `RHEOS/docs`. To this `Project.toml` we have added all the packages needed for the documentation: `Documenter.jl`, `Literate.jl`, `PyPlot.jl`, `PyCall.jl`. Of course, as we use RHEOS in the documentation, we also have to add this, but we add it using `Pkg.develop(path = "..")` so that it remains in sync with the version of RHEOS we are documenting. To build the documentation locally you should activate this environment so that it is tested in the same way it will be built remotely. Therefore, if you are in the `RHEOS/` directory, the correct command for building the documentation is `julia --project=docs/ docs/make.jl`.
<br><br>

### *Documenter.jl Generated Files*

Running `make.jl` using the command above will build the HTML documentation in `RHEOS/docs/build`. To view the documentation after it is built, go into the `build` directory and open up `index.html` in your browser. You should see the documentation in the same format as it exists on-line. When you click on a link in this page, it won't take you directly to that page, it takes you a directory structure page with only `index.html` present. You will need to click on `index.html` again. This is expected behaviour when inspecting locally. As long as clicking that `index.html` takes you to the link you were originally expecting to get to, everything is fine.

A last important point on the `RHEOS/docs/build` directory. It is listed in the `.gitignore` file which means it is not tracked by git. This is how it should be. The documentation seen online is built remotely and pushed to the gh-pages branch of the RHEOS repository. Therefore, there is no need for the local build folder to be tracked by git.
<br><br>

### *Modifying the Documentation*

The documentation of `Documenter.jl` and `Literate.jl` are the best resources for understanding how to modify the Julia and markdown files to meet your requirements. Both packages' docs are well written. However, a few specific aspects are dicussed below for convenience.

The documentation pages in `RHEOS/docs/src` that do not need to be interactive are written in markdown. Two nice features which are specific to the Documenter.jl parser, and used in the RHEOS documentation, are shown below.

For the RHEOS functions which have help docstrings above them, these can be conveniently imported into the documentation in an aesthetically pleasing way. This is achieved by the `@docs` macro. More specifically, see this excerpt from the `API.md` file:

    ```@docs
    resample
    cutting
    smooth
    extract
    ```

The resultant HTML file produced, assuming each of the `resample`, `cutting`, `smooth` and `extract` functions have docstrings, will be each function's argument signature, followed by its docstring.

Functions documented using the above mentioned `@docs` macro will automatically have a reference assigned to them, as will sections (and sub-headings if desired). This means that easily be linked to through use of the `@ref` macro. For example, from the `index.md` markdown file:

    The [API](@ref) section is a comprehensive list of RHEOS types and 
    functions, and brief descriptions of their use.

The word `API` would then be built as an HTML link to the `API` section of the docs in the resultant html. For more info on the `@ref` macro, including how to handle reference conflicts and other more complex cases, see the Documenter.jl documentation.
<br><br>

# Remote Documentation

As mentioned above, the actual on-line documentation is built remotely. This section is brief overview of how that process works. 

The best thing to do to understand this process is to try and familiarise yourself with the GitHub action file found in `RHEOS/.github/.workflows/Docbuild.yml`. When a commit or tag is pushed, or a pull request is merged, GitHub triggers the action defined in `DocBuild.yml`. The key part of this action is to run `make.jl` and deploy the documentation to the `gh-pages` branch of the repository.