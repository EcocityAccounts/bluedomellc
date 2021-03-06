---
title: usethis 1.6.0
author: Hadley Wickham
date: '2020-04-11'
slug: usethis-1-6-0
categories:
  - package
tags:
  - devtools
  - usethis
photo:
  author: Rock'n Roll Monkey
  url: https://unsplash.com/photos/R4WCbazrD1g
---



We're happy to announce that [usethis](https://usethis.r-lib.org) 1.6.0 is now available on CRAN. usethis is a package that facilitates interactive workflows for R project creation and development. It's mostly focussed on easing day-to-day package development, but many of its functions are also useful for non-package projects. Install usethis with:


```r
install.packages("usethis")
```

This blog post discusses three sets of improvements to usethis:

* New helpers for using GitHub Actions for continuous integration. If you're an 
  R package developer who uses GitHub, we recommend moving away from Travis and
  AppVeyor in favour of GitHub Actions.
  
* We've made a number of small tweaks to `create_package()` in order to reduce
  inessential friction in the initial startup phase of a package.
  
* We've continued to polish our tools for contributing to other people's
  packages through GitHub pull requests.
  
This release also includes a handful of new functions (my favourite is [`rename_file()`](https://usethis.r-lib.org/reference/rename_files.html)), many bug fixes, and lots of small improvements. We're slowly grinding down the rough edges, so usethis just works in more cases, and when it fails, it's more likely to give you error messages that help you quickly figure out the problem. As usual, you can find detailed notes about all these changes in the [change log](https://usethis.r-lib.org/news/index.html).

## GitHub Actions

[GitHub Actions](https://github.com/features/actions) is a continuous integration service that allows you to automatically run code whenever you push to GitHub. If you're developing a package this allows you to automate tasks like:

* Run `R CMD check` on multiple platforms (Linux, Windows, and Mac) and
  different versions of R (e.g., devel, release, oldrel).
* Record the code coverage of your unit tests.
* Re-build your [pkgdown](https://pkgdown.r-lib.org/) website.

Outside of a package, you can also use GitHub Actions to rebuild [blogdown](https://github.com/r-lib/actions/blob/master/examples/blogdown.yaml) and [bookdown](https://github.com/r-lib/actions/blob/master/examples/bookdown.yaml) sites, or regularly re-knit `.Rmd` files. 

Each GitHub Actions *workflow* is described in a yaml file stored in the `.github/workflows` directory of your repository. usethis v1.6.0 introduces new helper functions to create these files:

* [`use_github_action("check-standard")`](https://github.com/r-lib/actions/blob/master/examples/check-standard.yaml) runs `R CMD check` on the latest
  R release on Linux, Windows, and macOS, and R-devel on macOS. This ensures
  that your package works on all major operating systems, and alerts you to
  any potential problems in the next version of R.

* [`use_github_action("test-coverage")`](https://github.com/r-lib/actions/blob/master/examples/test-coverage.yaml) 
  uses [covr](http://covr.r-lib.org/) to measure the test coverage of your package 
  and publishes it to <http://codecov.io/>.
  
* [`use_github_action("pkgdown")`](https://github.com/r-lib/actions/blob/master/examples/pkgdown.yaml)
  uses [pkgdown](https://pkgdown.r-lib.org/) to build your package website and 
  publishes it to the `gh-pages` branch.

You can see examples of other workflows at <https://github.com/r-lib/actions/tree/master/examples>. The files in this directory are templates that you can easily copy into your package by running `use_github_action("name")`. We encourage you to look at the `.yaml` files that these functions create and customise them to meet your needs.

Compared to Travis-CI, GitHub Actions:

* Provides more resources, i.e. 6 hour build times and 20 concurrent builds 
  instead of 50 minute build times and ~5 concurrent builds. 
  
* Has more complete support for building on Windows and macOS and a more 
  natural way of using Docker containers. 
  
* Doesn't require any extra authentication because all code is run on GitHub.

* Is considerably easier to customise to provide workflows that we haven't
  made easy.

You can learn more by reading [_Github Actions with R_](https://ropenscilabs.github.io/actions_sandbox/), by Chris Brown, Murray Cadzow, Paula A Martinez, Rhydwyn McGuire, David Neuzerling, David Wilkinson, and Saras Windecker, or watching Jim Hester's [rstudio::conf(2020) presentation](https://resources.rstudio.com/rstudio-conf-2020/azure-pipelines-and-github-actions-jim-hester).

## Creating packages

Based on our experience teaching package development, we've made a few changes to how `create_package()` sets up a new package. The biggest difference is that we now assume that you're going to use roxygen2[^footnote] (although you can opt out with `roxygen = FALSE`), reducing some inconsistencies in development behaviours before and after your first run of `devtools::document()`:

[^footnote]: This seems like a reasonable assumption given that ~80% of new CRAN packages use roxygen2.

*   We automatically add a `RoxygenNote` field to the `DESCRIPTION`. This is a 
    subtle change that ensures `devtools::check()` re-documents your package even
    when you haven't yet run `devtools::document()`.
    
*   The default `NAMESPACE` no longer exports anything. This means that you
    must always use `@export` if you want functions to be available to the 
    end-user.

We made a couple of small changes to ease other minor frustrations:

*   `use_rstudio()` now sets the `LineEndingConversion` to `Posix` so that
    packages edited with RStudio always use LF line endings, regardless of 
    platform. This reduces spurious changes when multiple people collaborate
    on the same package.
  
*   The `usethis.description` option now lets you set `Authors@R = person()` 
    directly. That is, you can make an actual call to `person()` as opposed
    to writing a *string* that, when evaluated as code, returns a `person()`.
    This makes it less aggravating to detect and correct any mistakes. For
    example, in my `.Rprofile` I used to have:
    
    
    ```r
    options(usethis.description = list(
      `Authors@R` = 'person("Hadley", "Wickham", , "hadley@rstudio.com", role = c("aut", "cre"))'
    ))
    ```
    
    And now I have:

    
    ```r
    options(usethis.description = list(
      `Authors@R` = utils::person(
        "Hadley", "Wickham", , "hadley@rstudio.com",
        role = c("aut", "cre")
      )
    ))
    ```
      
    As you can see from the syntax highlighting, it's now much easier to see if 
    you've got all the quotes and commas in the right place. When you do this in
    `.Rprofile`, note that you **must** call it as `utils::person()`.

## Contributing to packages via GitHub pull requests

Based on our experiences at [tidyverse developer day](https://www.tidyverse.org/blog/2019/11/tidyverse-dev-day-2020/), we've tweaked the behaviour of usethis to ensure that new files have the same line ending as the rest of the project. We've also continued to polish our family of pull request helpers to work in more real-world situations. And, thanks to [Mine Cetinkaya-Rundel](http://www2.stat.duke.edu/~mc301/), we now have an article that [explains the overall workflow](https://usethis.r-lib.org/articles/articles/pr-functions.html).

## Thank you!

A big thanks to all 103 contributors who helped make this release happen via their contributions on GitHub. [&#x0040;aaronpeikert](https://github.com/aaronpeikert), [&#x0040;adelmofilho](https://github.com/adelmofilho), [&#x0040;Ahobert](https://github.com/Ahobert), [&#x0040;alandipert](https://github.com/alandipert), [&#x0040;andrie](https://github.com/andrie), [&#x0040;angela-li](https://github.com/angela-li), [&#x0040;antoine-sachet](https://github.com/antoine-sachet), [&#x0040;aosmith16](https://github.com/aosmith16), [&#x0040;atusy](https://github.com/atusy), [&#x0040;avalcarcel9](https://github.com/avalcarcel9), [&#x0040;barryrowlingson](https://github.com/barryrowlingson), [&#x0040;batpigandme](https://github.com/batpigandme), [&#x0040;boshek](https://github.com/boshek), [&#x0040;cderv](https://github.com/cderv), [&#x0040;Cervangirard](https://github.com/Cervangirard), [&#x0040;chsafouane](https://github.com/chsafouane), [&#x0040;coatless](https://github.com/coatless), [&#x0040;ColinFay](https://github.com/ColinFay), [&#x0040;CorradoLanera](https://github.com/CorradoLanera), [&#x0040;csgillespie](https://github.com/csgillespie), [&#x0040;cstepper](https://github.com/cstepper), [&#x0040;davechilders](https://github.com/davechilders), [&#x0040;davidchall](https://github.com/davidchall), [&#x0040;DavisVaughan](https://github.com/DavisVaughan), [&#x0040;dchiu911](https://github.com/dchiu911), [&#x0040;dragosmg](https://github.com/dragosmg), [&#x0040;edgararuiz](https://github.com/edgararuiz), [&#x0040;erindb](https://github.com/erindb), [&#x0040;espinielli](https://github.com/espinielli), [&#x0040;fabian-s](https://github.com/fabian-s), [&#x0040;fermumen](https://github.com/fermumen), [&#x0040;florianm](https://github.com/florianm), [&#x0040;fmichonneau](https://github.com/fmichonneau), [&#x0040;friep](https://github.com/friep), [&#x0040;gaborcsardi](https://github.com/gaborcsardi), [&#x0040;GegznaV](https://github.com/GegznaV), [&#x0040;GregorDeCillia](https://github.com/GregorDeCillia), [&#x0040;hadley](https://github.com/hadley), [&#x0040;igordot](https://github.com/igordot), [&#x0040;ijlyttle](https://github.com/ijlyttle), [&#x0040;IndrajeetPatil](https://github.com/IndrajeetPatil), [&#x0040;irenetlv](https://github.com/irenetlv), [&#x0040;isteves](https://github.com/isteves), [&#x0040;jdblischak](https://github.com/jdblischak), [&#x0040;jennybc](https://github.com/jennybc), [&#x0040;jimhester](https://github.com/jimhester), [&#x0040;jimmyg3g](https://github.com/jimmyg3g), [&#x0040;jmgirard](https://github.com/jmgirard), [&#x0040;JohnCoene](https://github.com/JohnCoene), [&#x0040;jplecavalier](https://github.com/jplecavalier), [&#x0040;jpritikin](https://github.com/jpritikin), [&#x0040;jrosen48](https://github.com/jrosen48), [&#x0040;jules32](https://github.com/jules32), [&#x0040;jzadra](https://github.com/jzadra), [&#x0040;karawoo](https://github.com/karawoo), [&#x0040;kevinushey](https://github.com/kevinushey), [&#x0040;kiernann](https://github.com/kiernann), [&#x0040;krlmlr](https://github.com/krlmlr), [&#x0040;lettucehead](https://github.com/lettucehead), [&#x0040;leungi](https://github.com/leungi), [&#x0040;lionel-](https://github.com/lionel-), [&#x0040;llrs](https://github.com/llrs), [&#x0040;lorenzwalthert](https://github.com/lorenzwalthert), [&#x0040;maelle](https://github.com/maelle), [&#x0040;MagdyLaban](https://github.com/MagdyLaban), [&#x0040;malcolmbarrett](https://github.com/malcolmbarrett), [&#x0040;Maschette](https://github.com/Maschette), [&#x0040;matthijsvanderloos](https://github.com/matthijsvanderloos), [&#x0040;maurolepore](https://github.com/maurolepore), [&#x0040;maxheld83](https://github.com/maxheld83), [&#x0040;MichaelChirico](https://github.com/MichaelChirico), [&#x0040;mikmart](https://github.com/mikmart), [&#x0040;MilesMcBain](https://github.com/MilesMcBain), [&#x0040;mine-cetinkaya-rundel](https://github.com/mine-cetinkaya-rundel), [&#x0040;mitchelloharawild](https://github.com/mitchelloharawild), [&#x0040;muschellij2](https://github.com/muschellij2), [&#x0040;nandriychuk](https://github.com/nandriychuk), [&#x0040;njtierney](https://github.com/njtierney), [&#x0040;noamross](https://github.com/noamross), [&#x0040;okhoma](https://github.com/okhoma), [&#x0040;overdodactyl](https://github.com/overdodactyl), [&#x0040;pachamaltese](https://github.com/pachamaltese), [&#x0040;pat-s](https://github.com/pat-s), [&#x0040;perezp44](https://github.com/perezp44), [&#x0040;petrbouchal](https://github.com/petrbouchal), [&#x0040;phargarten2](https://github.com/phargarten2), [&#x0040;pieterjanvc](https://github.com/pieterjanvc), [&#x0040;ramiromagno](https://github.com/ramiromagno), [&#x0040;riccardoporreca](https://github.com/riccardoporreca), [&#x0040;rich-iannone](https://github.com/rich-iannone), [&#x0040;Robinlovelace](https://github.com/Robinlovelace), [&#x0040;romainfrancois](https://github.com/romainfrancois), [&#x0040;rossellhayes](https://github.com/rossellhayes), [&#x0040;rundel](https://github.com/rundel), [&#x0040;ryapric](https://github.com/ryapric), [&#x0040;slyrus](https://github.com/slyrus), [&#x0040;smingerson](https://github.com/smingerson), [&#x0040;smwindecker](https://github.com/smwindecker), [&#x0040;strboul](https://github.com/strboul), [&#x0040;timtrice](https://github.com/timtrice), [&#x0040;TylerGrantSmith](https://github.com/TylerGrantSmith), [&#x0040;VincentGuyader](https://github.com/VincentGuyader), and [&#x0040;wch](https://github.com/wch).
