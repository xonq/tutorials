# A compilation of ggplot2 commands and usage info

## NOTE
I attempted to keep optional additions in the `Label additions` section. Required additions are included w/the command.
Keep in mind the `+` needs to be appended to the line you are adding the command to.

## dataframe transformations
### melt - `reshape2` package
`melted = melt($COLUMN, id.vars = "$COLUMN" )`
#### subset melted df
`submelted = melted[melted$COLUMn %in% c('$LABEL1', '$LABELn', ...), ]`


## Label additions
### label points as text
`+	geom_text( aes( label = ifelse( index == '$SUBPHYLUM', as.character( $NAME ), '')), hjust=0, vjust=0 )`
### vertical axis label
`+	theme( axis.text.x = element_text( angle = 90, hjust = 1, vjust = 0.5 ))`
### trendlines
`+	geom_smooth( method = lm, se = FALSE, fullrange = TRUE )`


## scatter plots
### basic scatter plot
`ggplot($DATA, aes( x = $X, y = $Y, color = $COLOR )) + geom_point()`
#### trendlines
`+	geom_smooth( method = lm, se = FALSE, fullrange = TRUE )`
#### trendlines w/confidence intervals
`+	geom_smooth( method = lm, se = TRUE, fullrange = TRUE )`


## bar plots
Bar plots often have to use a "melted" dataset. This requires `reshape2` package installed and loaded.
### basic bar plot
`ggplot( melted, aes( $COLUMN, value, fill = variable )) + geom_bar( position = "fill", stat = "identity")`
#### percent bar plot
`+	scale_y_continuous( labels = percent )`
#### stacked bar plot
`+	geom_bar(position = "stack", stat = "identity" )`
#### dodge bar plot
`+	geom_bar(position = "dodge", stat = "identity" )`


## violin plot
`ggplot( df, aes( x = $X, y = $Y )) + geom_violin( width = $WIDTH )`


## heatmap
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
