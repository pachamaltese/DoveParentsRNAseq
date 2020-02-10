Plots with Prolactin
====================

    library(tidyverse)

    ## ── Attaching packages ─────────────────────────────────────────────────────────── tidyverse 1.3.0 ──

    ## ✓ ggplot2 3.2.1     ✓ purrr   0.3.3
    ## ✓ tibble  2.1.3     ✓ dplyr   0.8.3
    ## ✓ tidyr   1.0.0     ✓ stringr 1.4.0
    ## ✓ readr   1.3.1     ✓ forcats 0.4.0

    ## ── Conflicts ────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

    library(cowplot)

    ## 
    ## Attaching package: 'cowplot'

    ## The following object is masked from 'package:ggplot2':
    ## 
    ##     ggsave

    library(magick)

    ## Linking to ImageMagick 6.9.9.39
    ## Enabled features: cairo, fontconfig, freetype, lcms, pango, rsvg, webp
    ## Disabled features: fftw, ghostscript, x11

    library(png)
    library(grid)
    library(ggimage)

    ## 
    ## Attaching package: 'ggimage'

    ## The following object is masked from 'package:cowplot':
    ## 
    ##     theme_nothing

    library(apaTables)

    source("../R/themes.R") 
    source("../R/functions.R")
    source("../R/icons.R")

    ## Warning: Column `icons` joining factor and character vector, coercing into
    ## character vector

    knitr::opts_chunk$set(fig.path = '../figures/',message=F, warning=FALSE)

Circulating levels of prolactin and prolactin expression
--------------------------------------------------------

    prolactin <- read_csv("../results/07_hormones.csv") %>%
        filter(study == "characterization", hormone %in% c("prolactin"))  %>% 
        droplevels() 
    prolactin$treatment <- factor(prolactin$treatment, levels = alllevels)

    vsd_path <- "../results/DEseq2/"   # path to the data
    vsd_files <- dir(vsd_path, pattern = "*vsd.csv") # get file names
    vsd_pathfiles <- paste0(vsd_path, vsd_files)
    vsd_files

    ## [1] "female_gonad_vsd.csv"        "female_hypothalamus_vsd.csv"
    ## [3] "female_pituitary_vsd.csv"    "male_gonad_vsd.csv"         
    ## [5] "male_hypothalamus_vsd.csv"   "male_pituitary_vsd.csv"

    PRLvsd <- vsd_pathfiles %>%
      setNames(nm = .) %>% 
      map_df(~read_csv(.x), .id = "file_name")  %>% 
      dplyr::rename("gene" = "X1") %>% 
      pivot_longer(cols = L.G118_female_gonad_control:y98.o50.x_male_pituitary_inc.d3, 
                   names_to = "samples", values_to = "counts") %>%
      filter(gene == "PRL")  %>%
      dplyr::mutate(sextissue = sapply(strsplit(file_name, '_vsd.csv'), "[", 1)) %>%
      dplyr::mutate(sextissue = sapply(strsplit(sextissue, '../results/DEseq2/'), "[", 2)) %>%
      dplyr::mutate(sex = sapply(strsplit(sextissue, '\\_'), "[", 1),
                    tissue = sapply(strsplit(sextissue, '\\_'), "[", 2),
                    treatment = sapply(strsplit(samples, '\\_'), "[", 4)) %>%
      dplyr::mutate(treatment = sapply(strsplit(treatment, '.NYNO'), "[", 1)) %>%
      dplyr::select(sex, tissue, treatment, gene, samples, counts)

    PRLvsd$treatment <- factor(PRLvsd$treatment, levels = alllevels)

    PRLhyp <- PRLvsd %>% filter(tissue == "hypothalamus")
    PRLpit <- PRLvsd %>% filter(tissue == "pituitary")
    PRLgon <- PRLvsd %>% filter(tissue == "gonad")

    plotprolactin <- function(df, myy, myylab, mysubtitle ){
      
      p <-  ggplot(df, aes(x = treatment, y = myy)) +
        geom_boxplot(aes(fill = treatment, color = sex)) +
        theme_B3() +
        scale_fill_manual(values = allcolors) +
        scale_color_manual(values = sexcolors) +
        labs(y = "prolactin (ng/mL)", x = NULL) +
        theme(legend.position = c(0.85,0.15), legend.direction = "horizontal") + 
      labs(x = "parental stage", subtitle = mysubtitle, y= myylab) +
          guides(fill = guide_legend(nrow = 1)) 
      return(p)
    }


    a <- plotprolactin(PRLhyp, PRLhyp$counts, "PRL", "hypothalamus") + 
      theme(legend.position = "none", axis.text.x = element_blank(), axis.title.x = element_blank(),
            axis.title.y = element_text(face = "italic"))
    b <- plotprolactin(PRLpit, PRLpit$counts, "PRL", "pituitary") + 
      theme(legend.position = "none", axis.text.x = element_blank(), axis.title.x = element_blank(),
            axis.title.y = element_text(face = "italic"))
    c <- plotprolactin(prolactin, prolactin$plasma_conc, "prolactin (ng/mL)", "blood") +
      theme(legend.position = "none", axis.text.x = element_text(angle = 45, hjust = 1)) 
    d <- plotprolactin(PRLgon, PRLgon$counts, "PRL", "gonads") + 
      theme(legend.position = "none", axis.text.x = element_text(angle = 45, hjust = 1),
            axis.title.y = element_text(face = "italic"))

    mylegend <- plotprolactin(PRLvsd, PRLvsd$counts, "PRL", "gonads") +
      theme(legend.position = "bottom",
            legend.direction = "vertical",
            legend.key.size = unit(0.5, 'lines')) +
      guides(color = guide_legend(ncol = 1),
             shape = guide_legend(ncol = 1))
    mylegend <- get_legend(mylegend)

    allPLRplots <- plot_grid(a,b,c,d, labels = "auto", rel_heights = c(0.425,0.575))

    fig3 <- plot_grid(allPLRplots, mylegend , nrow = 2, rel_heights = c(1,0.2))
    fig3

![](../figures/fig3-1.png)