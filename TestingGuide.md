# **RHEOS Testing - A Brief Overview**

This document is a short guide into how the RHEOS testing procedure works.

The test creation and running process can be summarised by the following. Tests are written in the file `RHEOS/test/runtests.jl`. They consist of code blocks that return `true` or `false` depending on whether or not the test has passed, and these code blocks are prefaced by the `@test` macro. For readability, tests longer than one line of code should be encapsulated in functions. For organisational benefit, the tests are generally split off into one test file per source file, with the test files being included from the main `runtests.jl` script. The tests can then be run locally, and will automatically run remotely after running `git push`.
<br><br>

# Local Testing

### *Directory Structure*

Everything testing related lives in the `RHEOS/tests` folder. Note that in the test folder there is one file per file in `RHEOS/src` and an additional file called `runtests.jl`. Having one test file per source file is purely for organisational purposes. The only file (and filename) of special significance is the `runtests.jl` file. This is the Julia file that will be run when we type `test RHEOS` in the Julia REPL's `pkg` mode. It is also the file that is run by the GitHub actions `RHEOS/.github/.workflows/NightlyTests.yml` and `RHEOS/.github/.workflows/StableTests.yml` on their Linux and Windows servers when we push a commit or tag, or merge a pull request to the GitHub repository.
<br><br>

### *Writing a Test*

Ideally, each function in the source code should have at least one test. For clear organisation, any test that requires more than one simple line of code should be written as a function. The line of code/function test should return a boolean value of `true` or `false` depending on whether the test passed or not. Then the line of code/function call should be prefaced with an `@test` macro. Let's take a look at an example:

    function _derivCD(tol)
        x = Vector(0.0:0.001:1.5)
        y = x.^2
        dy = 2*x
        dy_numeric = RHEOS.derivCD(y, x)
        isapprox(dy_numeric, dy, atol=tol)
    end
    @test _derivCD(tol)

This function tests the central difference derivative function in RHEOS. Notice that although it is an approximate test it still returns a discrete boolean true/false value indicating whether or not the result is within a prescribed absolute tolerance. (In this case the `tol` used is defined as a global constant earlier in the source file.)
<br><br>

### *Running Tests Locally*

To run tests locally you can either go into `pkg` mode and type `test RHEOS`, or you can navigate into the `RHEOS/tests` directory, open up a Julia REPL and type `include("runtests.jl")`. If a test fails, an error will be generated.
<br><br>

# Remote Testing

The RHEOS github repository is set-up with GitHub actions to test on Linux (latest version of stable Julia, and nightly build of Julia) and Windows (just latest stable Julia version) servers respectively. If any of the tests fail, this will be indicated on the appropriate badge in the github repository readme page.

If all tests pass locally, but fail on GitHub actions you will need to investigate. To do this, you can click on the badge in the `readme.md` which is failing. Click on the commit name that is failing, and then the job that is failing (on the left hand bar). You should then be able to inspect the terminal output from the server to determine the line at which the tests failed, and try to debug.

For a more detailed understanding of the remote testing, you should inspect the GitHub action files `RHEOS/.github/.workflows/NightlyTests.yml` and `RHEOS/.github/.workflows/StableTests.yml`.