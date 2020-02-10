    source("../R/wrangledata.R")

    ## Warning: Missing column names filled in: 'X1' [1]

PCA
---

    subsetmakepca <- function(whichtissue, whichtreatment, whichsex){   
      colData <- colData %>%    
        dplyr::filter(tissue %in% whichtissue,  
                      treatment %in% whichtreatment,    
                      sex %in% whichsex)    
      row.names(colData) <- colData$V1  
      # save counts that match colData  
      savecols <- as.character(colData$V1)  
      savecols <- as.vector(savecols)   
        
      countData <- as.data.frame(t(countData))  
      countData <- countData %>% dplyr::select(one_of(savecols))    
      countData <- as.data.frame(t(countData))  
        
      mypca <- prcomp(countData)    
      mypcadf <- data.frame(PC1 = mypca$x[, 1], PC2 = mypca$x[, 2], PC3 = mypca$x[, 3],     
                      PC4 = mypca$x[, 4],PC5 = mypca$x[, 5],PC6 = mypca$x[, 6], 
                      ID = row.names(countData))    
      mypcadf$V1 <- row.names(mypcadf)  
      mypcadf <- left_join(colData, mypcadf)    
      mypcadf <- mypcadf %>% dplyr::select(bird,sex,tissue,treatment,PC1:PC6)   
      return(mypcadf)   
    }   


    plotcolorfulpcs <- function(mypcadf,  whichfactor, whichcolors){    
      p <- mypcadf %>%  
        ggplot(aes(x = PC1, y = PC2, shape = tissue, color = whichfactor )) +   
        geom_point(size = 1)  + 
        theme_B3() +    
        theme(legend.title = element_blank(),   
             axis.text = element_blank(),   
             legend.position = "none") +    
        labs(x = "PC1", y = "PC2")  +   
        scale_color_manual(values = whichcolors) +  
        scale_shape_manual(values = myshapes)   +
        stat_ellipse( )
      
      return(p) 
    }   


    makefvizdf <-  function(whichtissue, whichtreatment, whichsex){ 
      colData <- colData %>%    
          dplyr::filter(tissue %in% whichtissue,    
                      treatment %in% whichtreatment,    
                      sex %in% whichsex)    
      row.names(colData) <- colData$V1  
      # save counts that match colData  
      savecols <- as.character(colData$V1)  
      savecols <- as.vector(savecols)   
        
      countData <- as.data.frame(t(countData))  
      countData <- countData %>% dplyr::select(one_of(savecols))    
      countData <- as.data.frame(t(countData))  
      mypca <- prcomp(countData)    
      return(mypca) 
    }   

    plotfriz <- function(frizdf){
      
      p <- fviz_pca_var(frizdf,  labelsize = 3.5 , axes.linetype = "blank", 
                        repel = T , select.var= list(contrib = 3), col.var = "black")  + 
        labs(title = NULL) + 
        theme_B3() + 
        theme(axis.text = element_blank())
      return(p)
    }

    fpca <- subsetmakepca(tissuelevels, charlevels, "female")   

    ## Warning: Column `V1` joining factor and character vector, coercing into
    ## character vector

    mpca <- subsetmakepca(tissuelevels, charlevels, "male") 

    ## Warning: Column `V1` joining factor and character vector, coercing into
    ## character vector

    ffviz <- makefvizdf(tissuelevels, charlevels, "female") 
    mfviz<- makefvizdf(tissuelevels, charlevels, "male")    

    f1 <- plotcolorfulpcs(fpca,fpca$treatment, allcolors) + labs(subtitle = "females")
    f2 <- plotfriz(ffviz) + labs(subtitle = " ")

    m1 <- plotcolorfulpcs(mpca,mpca$treatment, allcolors) + labs(subtitle = "males")
    m2 <- plotfriz(mfviz) + labs(subtitle = " ")


    allpcaplots <- plot_grid(f1,m1,f2,m2, nrow = 2, rel_heights = c(0.6,0.4), 
                    labels= c("a"," ", "b"), label_size = 12)

    ## Warning in MASS::cov.trob(data[, vars]): Probable convergence failure

    ## Warning in MASS::cov.trob(data[, vars]): Probable convergence failure

    ## Warning in MASS::cov.trob(data[, vars]): Probable convergence failure

    ## Warning in MASS::cov.trob(data[, vars]): Probable convergence failure

    forlegend <- plotcolorfulpcs(mpca,mpca$treatment, allcolors) + 
      theme(legend.position = "left",
            legend.direction = "vertical",
            legend.key.size = unit(0.5, 'lines')) +
      guides(color = guide_legend(ncol = 1),
             shape = guide_legend(ncol = 1))
    mylegend <- get_legend(forlegend)

    ## Warning in MASS::cov.trob(data[, vars]): Probable convergence failure

    ## Warning in MASS::cov.trob(data[, vars]): Probable convergence failure

    supplefig1 <- plot_grid(allpcaplots, mylegend, nrow = 1, rel_widths = c(1,0.2))
    supplefig1

![](../figures/supplfig1-1.png)