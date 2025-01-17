
<!-- README.md is generated from README.Rmd. Please edit that file -->

# {diades.atlas}

<!-- badges: start -->

[![R build
status](https://github.com/inrae/diades.atlas/workflows/R-CMD-check-docker-renv/badge.svg)](https://github.com/inrae/diades.atlas/actions)
[![Codecov test
coverage](https://codecov.io/gh/inrae/diades.atlas/branch/main/graph/badge.svg)](https://codecov.io/gh/inrae/diades.atlas?branch=main)
<!-- badges: end -->

A Shiny application to explore data.

# Installation

``` r
remotes::install_github('inrae/diades.atlas')
```

# Websites for validations :

-   Package documentation : <https://inrae.github.io/diades.atlas/>
-   Package documentation (development version) :
    <https://inrae.github.io/diades.atlas/dev/>
-   Shiny App : <https://connect.thinkr.fr/diadesatlasui18n/>

# Translation

Les instructions pour préparer la traduction sont dans
“dev/translation.Rmd”.  
Trois modes de traduction cohabitent:

-   Traductions présentes dans la base de données
-   Traductions à compléter dans des fichiers Markdown sous forme de
    Pull Request sur le projet
-   Traductions à compléter dans un fichier Google Sheet partagé

Dans tous les cas, les développeurs devront suivre le contenu de
“dev/translation.Rmd” lors de chaque mises à jour.

# Backend requirement

## The apps needs to connect to a db.

It can be set via :

``` r
golem::amend_golem_config(
  "POSTGRES_DBNAME",
  "diades"
)
golem::amend_golem_config(
  "POSTGRES_HOST",
  "localhost"
)
golem::amend_golem_config(
  "POSTGRES_PORT",
  5432
)
Sys.setenv("POSTGRES_USER" = "diadesatlas_owner")
Sys.setenv("POSTGRES_PASS" = "thinkrpassword")
```

For dev purpose, please refer to the doc in the diadesdata repo on
ThinkR’s forge (restricted access).

Verify connexion works

``` r
pkgload::load_all()
session <- new.env()
connect(session)
con <- get_con(session)

DBI::dbListTables(con)
DBI::dbDisconnect(con)
```

## Dev - Mongo

The app requires a Mongo database. For dev, you need to run the
following:

    docker run --name=mongo --rm -p 2811:27017 -e MONGO_INITDB_ROOT_USERNAME=Colin -e MONGO_INITDB_ROOT_PASSWORD=AsAboveSoBelow789123 mongo:4.0

Stop it at the end

    docker kill mongo

# Dev - vignettes

Vignettes are compiled manually by developers. Raw vignettes are stored
in “data-raw”. Instructions to compile them are in the vignette itself.

However, they can all be prepared from this script:

``` r
remotes::install_local(upgrade = "never")

file.copy(here::here("dev", "translation.Rmd"),
          here::here("data-raw", "translation.Rmd"))

all_vignettes <- c("aa-data-exploration-and-preparation",
                   "bb-page1-catch-bycatch",
                   "bc-page2-present",
                   "bd-page3-climate-change",
                   "be-page4-future", 
                   "translation")

for (vignette_name in all_vignettes) {
  # vignette_name <- "aa-data-exploration-and-preparation"
  vignette_file <- paste0(vignette_name, ".Rmd")
  
  rmarkdown::render(
    input = here::here(file.path("data-raw", vignette_file)),
    output_format = "rmarkdown::html_vignette",
    output_options = list(toc = TRUE),
    output_file = here::here(file.path("vignettes", vignette_file))
  )
  
  # Add header for title
  lines <- readLines(here::here(file.path("vignettes", vignette_file)))
  
  cat(
    glue::glue('---
title: ".{vignette_name}."
output: rmarkdown::html_vignette
vignette: >
  %\\VignetteIndexEntry{.{vignette_name}.}
  %\\VignetteEngine{knitr::rmarkdown}
  %\\VignetteEncoding{UTF-8}
---
', .open = ".{", .close = "}."),
lines,
sep = "\n", 
file = here::here(file.path("vignettes", vignette_file))
  )
}
file.remove(here::here("data-raw", "translation.Rmd"))
```
