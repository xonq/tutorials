# A compilation of R/ggplot2/gg3d commands and usage info
<br/>

## NOTE
I attempted to keep optional additions in the `Label additions` section. Required additions are included w/the command.
Keep in mind the `+` needs to be appended to the line you are adding the command to.


<br/><br/><br/>
## INSTALLING PACKAGES
### ggplot2 example
`install.packages('ggplot2')`
### gg3d (github) example - allows 3D ggplot2 extension
```
install.packages('devtools')
library('devtools')
devtools::install_github('AckerDWM/gg3D')
```

<br/><br/>
## TRANSFORMATIONS
### melt - `reshape2` package
`melted = melt($COLUMN, id.vars = "$COLUMN" )`
#### subset melted df
`submelted = melted[melted$COLUMn %in% c('$LABEL1', '$LABELn', ...), ]`

<br/><br/>
## LABEL ADDITIONS
### label points as text
`+	geom_text( aes( label = ifelse( index == '$SUBPHYLUM', as.character( $NAME ), '')), hjust=0, vjust=0 )`
### vertical axis label
`+	theme( axis.text.x = element_text( angle = 90, hjust = 1, vjust = 0.5 ))`
### trendlines
`+	geom_smooth( method = lm, se = FALSE, fullrange = TRUE )`

<br/><br/>
## SCATTER PLOTS
### basic scatter plot
`ggplot($DATA, aes( x = $X, y = $Y, color = $COLOR )) + geom_point()`
#### trendlines
`+	geom_smooth( method = lm, se = FALSE, fullrange = TRUE )`
#### trendlines w/confidence intervals
`+	geom_smooth( method = lm, se = TRUE, fullrange = TRUE )`

<br/><br/>
## BAR PLOTS
Bar plots often have to use a "melted" dataset. This requires `reshape2` package installed and loaded.
### basic bar plot
`ggplot( melted, aes( $COLUMN, value, fill = variable )) + geom_bar( position = "fill", stat = "identity")`
#### percent bar plot
`+	scale_y_continuous( labels = percent )`
#### stacked bar plot
`+	geom_bar(position = "stack", stat = "identity" )`
#### dodge bar plot
`+	geom_bar(position = "dodge", stat = "identity" )`

<br/><br/>
## VIOLIN PLOTS
`ggplot( df, aes( x = $X, y = $Y )) + geom_violin( width = $WIDTH )`


<br/><br/><br/>
## HEATMAP
There is some finessing involved with specifying column names etc. The dataframe here has row names in the first column and column names as the column names in the dataframe.
### preparing data and plotting
```
data = tsv[,-1]
rownames(data) = tsv[,1]
heatmap(as.matrix(data), scale="none")
```
### no dendogram or reordering
`heatmap(as.matrix(data), Colv = NA, Rowv = NA, scale="none")` # can specify `column` in scale to make relative to column
### colors
`heatmap(as.matrix(data1), scale="none", Colv=NA, Rowv=NA, col= colorRampPalette(brewer.pal(9, "Reds"))(256))`


<br/><br/><br/>
## ORDINATIONS
### NMDS
#### preparing data (2D)
```
library(vegan)
com = metabolicCounts[,2:20] # range of columns that have values
m_com = as.matrix(com)
nmds = metaMDS(m_com, distance="euclidean", k=2) # distance can be changed to your use case, k = dimensions
data.scores = as.data.frame(scores(nmds))
data.scores$INDEX = metabolicCounts$INDEX
```
#### optional: adding the fit
##### if it is a categorical variable, plot as points and substitute 'vectors' for 'factors'
```
mds.fit = envfit( nmds, com, permutations = 999, na.rm = TRUE )
mds_coord_cont = as.data.frame(scores(mds.fit, "vectors") * ordiArrowMul(mds.fit))
```
#### plotting the data
```
library(ggplot2)
colors = c(brewer.pal(name="Dark2", n = 8), brewer.pal(name="Paired", n = 6))
xx = ggplot(data.scores, aes(x = NMDS1, y = NMDS2)) + geom_text( aes( label = as.character( ome ), colour = sphy)) + scale_color_manual(values = colors)
```
##### with the fit
```
xx = ggplot(data.scores, aes(x = NMDS1, y = NMDS2)) + geom_text( aes( label = as.character( ome ), colour = sphy)) + scale_color_manual(values = colors) + geom_segment(aes(x = 0, y= 0, xend = NMDS1, yend = NMDS2), data = mds_coord_cat, size =1, alpha = 0.5, colour = "grey30") + geom_text(data = mds_coord_cat, aes( x = NMDS1, y = NMDS2), label = row.names(mds_coord_cat), colour = "grey10", fontface = "bold")
```
<br/>

#### preparing (3D)
```
library('vegan3d')
com = metabolicCounts[,2:20]
m_com = as.matrix(com)
nmds3 = metaMDS(m_com, distance='euclidean', k=3)
data.scores = as.data.frame(scores(nmds3))
data.scores$INDEX = metabolicCounts$INDEX
```

<br/>

#### plotting (3D - fit not available yet)
```
plot_ly( x = data.scores$NMDS1, y = data.scores$NMDS2, z = data.scores$NMDS3, color = data.scores$sphy, colors = colors )
```
