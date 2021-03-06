LDA analysis of characterization and manipulation
-------------------------------------------------

    colData <- read.csv("../metadata/00_samples.csv", row.names = 1)

    charHyp <- colData %>% filter(study == "charcterization", tissue == "hypothalamus") %>% droplevels()
    charPit <- colData %>% filter(study == "charcterization", tissue == "pituitary") %>% droplevels()
    charGon <- colData %>% filter(study == "charcterization", tissue == "gonad") %>% 
      mutate(tissue = fct_recode(tissue, "gonads" = "gonad")) %>% droplevels()

    manipHyp <- colData %>% filter(study == "manipulation", tissue == "hypothalamus") %>% droplevels()
    manipPit <- colData %>% filter(study == "manipulation", tissue == "pituitary") %>% droplevels()
    manipGon <- colData %>% filter(study == "manipulation", tissue == "gonad") %>% 
      mutate(tissue = fct_recode(tissue, "gonads" = "gonad")) %>% droplevels()


    selectvsd <- function(pathtofile, colData){
      
      df <- read.csv(pathtofile, row.names = 1)
      savecols <- as.character(colData$V1) 
      savecols <- as.vector(savecols) 
      df <- df %>% dplyr::select(one_of(savecols)) 
      
      # keep only 100 genes fornow
      #df <- head(df, 100)
      
      df <- as.data.frame(t(df))
      df$V1 <- row.names(df)
      return(df)
    }

    vsd.hyp.train <- selectvsd("../results/06_hypallvsd.csv",  charHyp)
    vsd.pit.train <- selectvsd("../results/06_pitallvsd.csv",  charPit)
    vsd.gon.train <- selectvsd("../results/06_gonallvsd.csv",  charGon)

    vsd.hyp.test <- selectvsd("../results/06_hypallvsd.csv",  manipHyp)
    vsd.pit.test <- selectvsd("../results/06_pitallvsd.csv",  manipPit)
    vsd.gon.test <- selectvsd("../results/06_gonallvsd.csv",  manipGon)

Linear discriminant analysis (LDA)
----------------------------------

<a href="http://www.sthda.com/english/articles/36-classification-methods-essentials/146-discriminant-analysis-essentials-in-r/" class="uri">http://www.sthda.com/english/articles/36-classification-methods-essentials/146-discriminant-analysis-essentials-in-r/</a>

    LDanalysis <- function(trainsamples, traindata, testdata, testsamples){
      
      train.data <- left_join(trainsamples, traindata) %>%
        dplyr::select(-V1, -bird, -tissue, -group, -study)
      
      test.data <- left_join(testsamples, testdata) %>%
        dplyr::select(-V1, -bird, -tissue, -group, -study, -treatment)
      
      testsamples <- testsamples
      
      # Normalize the data. Categorical variables are automatically ignored.
      # Estimate preprocessing parameters
      preproc.param <- train.data %>% 
        preProcess(method = c("center", "scale"))
      
      # Transform the data using the estimated parameters
      train.transformed <- preproc.param %>% predict(train.data)
      test.transformed <- preproc.param %>% predict(test.data)
      
      # LDA analysis
      # Fit the model
      model <- lda(treatment ~ ., data = train.transformed)
      # Make predictions
      predictions <- model %>% predict(test.transformed)
      
      # Model accuracy
      print("model accuracy")
      #print("predictions$class==test.transformed$treatment)")
     # print(mean(predictions$class==test.transformed$treatment))
      
      # results
      print("the samples sizes")
      print(model$counts)
      
      print("the prior probabilities used")
      print(model$prior)
      
      print("svd: the singular values, which give the ratio of the between- and within-group standard deviations on the linear discriminant variables. Their squares are the canonical F-statistics.")
      print(model$svd)
      
      
      #  predictions
      predictions <- model %>% predict(test.transformed)
      head(predictions)
      
      # Predicted classes
      #print(predictions$class, 6)
      # Predicted probabilities of class memebership.
      #print(predictions$posterior, 6) 
      # Linear discriminants
      #print(predictions$x, 3)
      
      
      predictedstage <-  predict(model, test.transformed)$class
      testsamples$predictedstage <- predictedstage
      
      lda.data <- cbind(testsamples, predictions$x)
      
      return(lda.data)
    }  

    LDA.hyp <- LDanalysis(charHyp, vsd.hyp.train, vsd.hyp.test, manipHyp)

    FALSE [1] "model accuracy"
    FALSE [1] "the samples sizes"
    FALSE    bldg control   hatch inc.d17  inc.d3  inc.d9     lay      n5      n9 
    FALSE      20      22      20      22      20      23      20      20      22 
    FALSE [1] "the prior probabilities used"
    FALSE      bldg   control     hatch   inc.d17    inc.d3    inc.d9       lay 
    FALSE 0.1058201 0.1164021 0.1058201 0.1164021 0.1058201 0.1216931 0.1058201 
    FALSE        n5        n9 
    FALSE 0.1058201 0.1164021 
    FALSE [1] "svd: the singular values, which give the ratio of the between- and within-group standard deviations on the linear discriminant variables. Their squares are the canonical F-statistics."
    FALSE [1] 12.290093  3.737176  3.211662  2.808772  2.549954  2.426908  2.113342
    FALSE [8]  2.026581

    LDA.pit <- LDanalysis(charPit, vsd.pit.train, vsd.pit.test, manipPit)

    FALSE [1] "model accuracy"
    FALSE [1] "the samples sizes"
    FALSE    bldg control   hatch inc.d17  inc.d3  inc.d9     lay      n5      n9 
    FALSE      20      25      20      22      20      24      20      20      22 
    FALSE [1] "the prior probabilities used"
    FALSE      bldg   control     hatch   inc.d17    inc.d3    inc.d9       lay 
    FALSE 0.1036269 0.1295337 0.1036269 0.1139896 0.1036269 0.1243523 0.1036269 
    FALSE        n5        n9 
    FALSE 0.1036269 0.1139896 
    FALSE [1] "svd: the singular values, which give the ratio of the between- and within-group standard deviations on the linear discriminant variables. Their squares are the canonical F-statistics."
    FALSE [1] 11.062064  7.608646  4.442513  3.502451  3.462257  2.818555  2.652531
    FALSE [8]  2.179710

    LDA.gon <- LDanalysis(charGon, vsd.gon.train, vsd.gon.test, manipGon)

    FALSE [1] "model accuracy"
    FALSE [1] "the samples sizes"
    FALSE    bldg control   hatch inc.d17  inc.d3  inc.d9     lay      n5      n9 
    FALSE      20      26      20      22      20      24      20      20      22 
    FALSE [1] "the prior probabilities used"
    FALSE      bldg   control     hatch   inc.d17    inc.d3    inc.d9       lay 
    FALSE 0.1030928 0.1340206 0.1030928 0.1134021 0.1030928 0.1237113 0.1030928 
    FALSE        n5        n9 
    FALSE 0.1030928 0.1134021 
    FALSE [1] "svd: the singular values, which give the ratio of the between- and within-group standard deviations on the linear discriminant variables. Their squares are the canonical F-statistics."
    FALSE [1] 9.826635 4.643176 3.744735 3.117603 2.699703 2.362898 2.231785 2.112316

    LDA.hyp$treatment <- factor(LDA.hyp$treatment, levels = alllevels)
    LDA.hyp$predictedstage <- factor(LDA.hyp$predictedstage, levels = alllevels)

    LDA.pit$treatment <- factor(LDA.pit$treatment, levels = alllevels)
    LDA.pit$predictedstage <- factor(LDA.pit$predictedstage, levels = alllevels)

    LDA.gon$treatment <- factor(LDA.gon$treatment, levels = alllevels)
    LDA.gon$predictedstage <- factor(LDA.gon$predictedstage, levels = alllevels)


    #myshapes = c("hypothalamus" = 20,  "pituitary" = 17,  "gonads" = 15)

    a <- ggplot(LDA.hyp, aes(x = LD1, LD2, color = predictedstage)) + geom_point(shape = 20) + theme(legend.position = "none") + scale_color_manual(values = colorscharmaip) 
    b <- ggplot(LDA.hyp, aes(x = LD1, LD2, color = treatment)) + geom_point(shape = 20) + theme(legend.position = "none") + scale_color_manual(values = colorscharmaip)

    c <- ggplot(LDA.pit, aes(x = LD1, LD2, color = predictedstage)) + geom_point(shape = 17) + theme(legend.position = "none") + scale_color_manual(values = colorscharmaip)
    d <- ggplot(LDA.pit, aes(x = LD1, LD2, color = treatment)) + geom_point(shape = 17) + theme(legend.position = "none") + scale_color_manual(values = colorscharmaip)

    e <- ggplot(LDA.gon, aes(x = LD1, LD2, color = predictedstage)) + geom_point(shape = 15) + theme(legend.position = "none") + scale_color_manual(values = colorscharmaip)
    f <- ggplot(LDA.gon, aes(x = LD1, LD2, color = treatment)) + geom_point(shape = 15) + theme(legend.position = "none") + scale_color_manual(values = colorscharmaip)

    plot_grid(a,b,c,d,e,f, nrow = 3)

![](../figures/LDA/LDAplots-1.png)

    g <- ggplot(LDA.hyp, aes(x = LD1, LD2, shape = predictedstage, color = treatment)) + 
      geom_point( ) + theme_B3() +
      theme(legend.position = "none") + 
      scale_color_manual(values = colorscharmaip) + 
      guides(color = guide_legend(show = FALSE))  +
      labs(subtitle = "hypothalamus")

    h <- ggplot(LDA.pit, aes(x = LD1, LD2, shape = predictedstage, color = treatment)) + 
      geom_point( ) + theme_B3() +
      theme(legend.position = "none") + scale_color_manual(values = colorscharmaip)  + 
      guides(color = guide_legend(show = FALSE)) +
      labs(subtitle = "piuitary")

    i <- ggplot(LDA.gon, aes(x = LD1, LD2, shape = predictedstage, color = treatment)) + 
      geom_point( ) + theme_B3() +
      theme(legend.position = "right") + scale_color_manual(values = colorscharmaip)  + 
      #guides(color = guide_legend(show = FALSE)) +
      guides(shape = guide_legend(ncol = 3),
             color = guide_legend(ncol = 3)) +
      labs(subtitle = "gonads")

    gh <- plot_grid(g,h, nrow = 1)
    ghi <- plot_grid(gh, i, nrow = 2)
    ghi

![](../figures/LDA/LDAplots-2.png)

    plotfacetresults <- function(df, whichgroups, mysubstitle, myshape){
      
      p <- df %>%
        filter(treatment %in% whichgroups) %>%
        ggplot(aes(x = LD1, LD2,  color = predictedstage)) + 
      geom_point(shape = myshape ) + theme_B3() +
        stat_ellipse() +
      theme(legend.position = "none") + 
      scale_color_manual(values = colorscharmaip) + 
      guides(color = guide_legend(show = FALSE))  +
      labs(y = mysubstitle) + 
      facet_wrap(~treatment, nrow = 1)
      return(p)
    }

    j <- plotfacetresults(LDA.gon, levelstiming, "hypothalamus\n LD2", 20) + labs(x = NULL)
    k <- plotfacetresults(LDA.pit, levelstiming, "pituitary\n LD2", 17) + theme(strip.text = element_blank()) + labs(x = NULL)
    l <- plotfacetresults(LDA.gon, levelstiming, "gonads\n LD2", 15) + theme(strip.text = element_blank()) + 
      theme(legend.position = "bottom") +
      guides(color = guide_legend(nrow  = 1))

    plot_grid(j,k,l, nrow = 3, rel_heights = c(1.2,1,1.6), align = "v")

![](../figures/LDA/LDAplots-3.png)

    m <- plotfacetresults(LDA.gon, levelsremoval, "hypothalamus\n LD2", 20) + labs(x = NULL)
    n <- plotfacetresults(LDA.pit, levelsremoval, "pituitary\n LD2", 17) + theme(strip.text = element_blank()) + labs(x = NULL)
    o <- plotfacetresults(LDA.gon, levelsremoval, "gonads\n LD2", 15) + theme(strip.text = element_blank()) + 
      theme(legend.position = "bottom") +
      guides(color = guide_legend(nrow  = 1))

    plot_grid(m,n,o, nrow = 3, rel_heights = c(1.2,1,1.6), align = "v")

![](../figures/LDA/LDAplots-4.png)

    plotfacetresults2 <- function(df, whichgroups, mysubstitle, myshape){
      p <- df %>%
        filter(treatment %in% whichgroups) %>%
      ggplot(aes(x = treatment, fill = predictedstage)) +
      geom_bar(position = position_fill(reverse = F))  +
      facet_wrap(~sex) +
        theme_B3() +
      theme(legend.position = "none") +
      scale_fill_manual(values = colorscharmaip) +
      labs(y = mysubstitle)  +
        geom_hline(yintercept = 0.5,  size = 0.25, linetype = 2)
      return(p)
    }

    j <- plotfacetresults2(LDA.gon, levelstiming, "hypothalamus", 20) + labs(x = NULL) + 
      theme(axis.text.x = element_blank()) +
      theme(legend.position = "top") + guides(fill = guide_legend(nrow  = 1)) 
    k <- plotfacetresults2(LDA.pit, levelstiming, "pituitary", 17) + 
      theme(strip.text = element_blank()) + labs(x = NULL) + theme(axis.text.x = element_blank())
    l <- plotfacetresults2(LDA.gon, levelstiming, "gonads", 15) + theme(strip.text = element_blank()) + 
       labs(x = NULL)

    plot_grid(j,k,l, nrow = 3, rel_heights = c(1.6,1,1.2))

![](../figures/LDA/LDAplots-5.png)

    m <- plotfacetresults2(LDA.gon, levelsremoval, "hypothalamus", 20) + labs(x = NULL) + theme(axis.text.x = element_blank()) +
        theme(legend.position = "top")  +  guides(fill = guide_legend(nrow  = 1))
    n <- plotfacetresults2(LDA.pit, levelsremoval, "pituitary", 17) + theme(strip.text = element_blank()) + labs(x = NULL) + theme(axis.text.x = element_blank())
    o <- plotfacetresults2(LDA.gon, levelsremoval, "gonads", 15) +
      theme(strip.text = element_blank())  + labs(x = NULL)

    plot_grid(m,n,o, nrow = 3, rel_heights = c(1.6,1,1.2))

![](../figures/LDA/LDAplots-6.png)

    m <- plotfacetresults2(LDA.gon, alllevels, "hypothalamus", 20) + labs(x = NULL) + theme(axis.text.x = element_blank()) +
        theme(legend.position = "top")  +  guides(fill = guide_legend(nrow  = 1))
    n <- plotfacetresults2(LDA.pit, alllevels, "pituitary", 17) + theme(strip.text = element_blank()) + labs(x = NULL) + theme(axis.text.x = element_blank())
    o <- plotfacetresults2(LDA.gon, alllevels, "gonads", 15) +
      theme(strip.text = element_blank())  + labs(x = NULL) + theme(axis.text.x = element_text(angle = 45, hjust = 1))

    plot_grid(m,n,o, nrow = 3, rel_heights = c(1.6,1,1.2))

![](../figures/LDA/LDAplots-7.png)

    library(kableExtra)

    df1 <- LDA.hyp %>% distinct(treatment, predictedstage) %>%
      group_by(treatment) %>%
      summarize(predictedstages = str_c(predictedstage , collapse = ", "))  %>%
      mutate(tissue = "hypothalamus") 
    kable(df1)

<table>
<thead>
<tr>
<th style="text-align:left;">
treatment
</th>
<th style="text-align:left;">
predictedstages
</th>
<th style="text-align:left;">
tissue
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
n5, n9, inc.d3, inc.d9, inc.d17
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d8
</td>
<td style="text-align:left;">
n5, inc.d3, n9, inc.d9
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
bldg, n9, inc.d3, lay, inc.d17, inc.d9, n5
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d17
</td>
<td style="text-align:left;">
inc.d9, n9, inc.d3, n5, lay, hatch
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
inc.d3, inc.d9, n9, n5, inc.d17
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.n2
</td>
<td style="text-align:left;">
inc.d3, n9, n5, inc.d9
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
n9, inc.d3, n5, inc.d9, inc.d17
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
</tbody>
</table>

    df2 <- LDA.hyp %>% 
      group_by(treatment,predictedstage) %>%
      summarize(n = n()) %>%
      mutate(tissue = "hypothalamus")  %>%
      arrange(tissue, treatment, desc(n))
    kable(df2)

<table>
<thead>
<tr>
<th style="text-align:left;">
treatment
</th>
<th style="text-align:left;">
predictedstage
</th>
<th style="text-align:right;">
n
</th>
<th style="text-align:left;">
tissue
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
n5
</td>
<td style="text-align:right;">
9
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
6
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
inc.d3
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
inc.d9
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
inc.d17
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d8
</td>
<td style="text-align:left;">
inc.d3
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d8
</td>
<td style="text-align:left;">
inc.d9
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d8
</td>
<td style="text-align:left;">
n5
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d8
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
bldg
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
lay
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
inc.d9
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
inc.d3
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
inc.d17
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
n5
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d17
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
9
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d17
</td>
<td style="text-align:left;">
n5
</td>
<td style="text-align:right;">
6
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d17
</td>
<td style="text-align:left;">
lay
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d17
</td>
<td style="text-align:left;">
inc.d3
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d17
</td>
<td style="text-align:left;">
inc.d9
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d17
</td>
<td style="text-align:left;">
hatch
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
n5
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
inc.d3
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
inc.d9
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
inc.d17
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.n2
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
11
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.n2
</td>
<td style="text-align:left;">
n5
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.n2
</td>
<td style="text-align:left;">
inc.d3
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
m.n2
</td>
<td style="text-align:left;">
inc.d9
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
n5
</td>
<td style="text-align:right;">
7
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
7
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
inc.d3
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
inc.d9
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
inc.d17
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
hypothalamus
</td>
</tr>
</tbody>
</table>

    df3 <- LDA.pit %>% distinct(treatment, predictedstage) %>%
      group_by(treatment) %>%
      summarize(predictedstages = str_c(predictedstage , collapse = ", ")) %>%
      mutate(tissue = "pituitary")
    kable(df3)

<table>
<thead>
<tr>
<th style="text-align:left;">
treatment
</th>
<th style="text-align:left;">
predictedstages
</th>
<th style="text-align:left;">
tissue
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
lay, n9, inc.d3, bldg
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d8
</td>
<td style="text-align:left;">
lay, n9, inc.d3, inc.d9
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
bldg, n9, inc.d3
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d17
</td>
<td style="text-align:left;">
n9, lay, hatch
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
lay, n9, hatch, inc.d17, n5
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.n2
</td>
<td style="text-align:left;">
hatch, n9, lay
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
hatch, n9, n5, lay
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
</tbody>
</table>

    df4  <- LDA.pit %>% 
      group_by(treatment,predictedstage) %>%
      summarize(n = n()) %>%
      mutate(tissue = "pituitary") %>%
      arrange(tissue, treatment, desc(n))
    kable(df4)

<table>
<thead>
<tr>
<th style="text-align:left;">
treatment
</th>
<th style="text-align:left;">
predictedstage
</th>
<th style="text-align:right;">
n
</th>
<th style="text-align:left;">
tissue
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
lay
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
bldg
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
inc.d3
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d8
</td>
<td style="text-align:left;">
lay
</td>
<td style="text-align:right;">
12
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d8
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d8
</td>
<td style="text-align:left;">
inc.d9
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d8
</td>
<td style="text-align:left;">
inc.d3
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
bldg
</td>
<td style="text-align:right;">
7
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
7
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
inc.d3
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d17
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
19
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d17
</td>
<td style="text-align:left;">
lay
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d17
</td>
<td style="text-align:left;">
hatch
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
hatch
</td>
<td style="text-align:right;">
6
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
n5
</td>
<td style="text-align:right;">
6
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
inc.d17
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
lay
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.n2
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
18
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.n2
</td>
<td style="text-align:left;">
lay
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
m.n2
</td>
<td style="text-align:left;">
hatch
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
n5
</td>
<td style="text-align:right;">
13
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
hatch
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
lay
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
pituitary
</td>
</tr>
</tbody>
</table>

    df5 <- LDA.gon %>% distinct(treatment, predictedstage) %>%
      group_by(treatment) %>%
      summarize(predictedstages = str_c(predictedstage , collapse = ", ")) %>%
      mutate(tissue = "gonads")
    kable(df5)

<table>
<thead>
<tr>
<th style="text-align:left;">
treatment
</th>
<th style="text-align:left;">
predictedstages
</th>
<th style="text-align:left;">
tissue
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
lay, inc.d9, n9, n5, inc.d3, hatch, bldg
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d8
</td>
<td style="text-align:left;">
n5, inc.d3, n9, lay, hatch
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
n9, inc.d17, n5, hatch, inc.d9, bldg, inc.d3
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d17
</td>
<td style="text-align:left;">
n9, lay, n5
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
lay, n9, bldg, hatch, n5
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.n2
</td>
<td style="text-align:left;">
hatch, n9, lay, n5, bldg
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
n5, n9, inc.d17, lay, hatch, inc.d3
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
</tbody>
</table>

    df6 <- LDA.gon %>% 
      group_by(treatment,predictedstage) %>%
      summarize(n = n()) %>%
      mutate(tissue = "gonads") %>%
      arrange(tissue, treatment, desc(n))
    kable(df6)

<table>
<thead>
<tr>
<th style="text-align:left;">
treatment
</th>
<th style="text-align:left;">
predictedstage
</th>
<th style="text-align:right;">
n
</th>
<th style="text-align:left;">
tissue
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
lay
</td>
<td style="text-align:right;">
6
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
n5
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
inc.d3
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
bldg
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
inc.d9
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d3
</td>
<td style="text-align:left;">
hatch
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d8
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
6
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d8
</td>
<td style="text-align:left;">
lay
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d8
</td>
<td style="text-align:left;">
inc.d3
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d8
</td>
<td style="text-align:left;">
n5
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d8
</td>
<td style="text-align:left;">
hatch
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
n5
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
bldg
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
inc.d17
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
inc.d3
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
inc.d9
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d9
</td>
<td style="text-align:left;">
hatch
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d17
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d17
</td>
<td style="text-align:left;">
n5
</td>
<td style="text-align:right;">
7
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.inc.d17
</td>
<td style="text-align:left;">
lay
</td>
<td style="text-align:right;">
6
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
lay
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
n5
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
hatch
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
prolong
</td>
<td style="text-align:left;">
bldg
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.n2
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
7
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.n2
</td>
<td style="text-align:left;">
n5
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.n2
</td>
<td style="text-align:left;">
lay
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.n2
</td>
<td style="text-align:left;">
hatch
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
m.n2
</td>
<td style="text-align:left;">
bldg
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
n5
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
lay
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
n9
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
inc.d17
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
inc.d3
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
<tr>
<td style="text-align:left;">
extend
</td>
<td style="text-align:left;">
hatch
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
gonads
</td>
</tr>
</tbody>
</table>
