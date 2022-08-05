# R wisdom
<br/>

## NOTE
I attempted to keep optional additions in the `Label additions` section. Required additions are included w/the command.
Keep in mind the `+` needs to be appended to the line you are adding the command to.

<br />

## COLOR PALETTES
```
colors <- c(
    "#000000","#004949","#009292","#ff6db6","#ffb6db",
    "#490092","#006ddb","#b66dff","#6db6ff","#b6dbff",
    "#920000","#924900","#db6d00","#24ff24","#ffff6d"
    )
```
```
colors <- c(
    "#F0A3FF", "#0075DC", "#993F00", "#4C005C", "#191919",
    "#005C31", "#2BCE48", "#FFCC99", "#808080", "#94FFB5",
    "#8F7C00", "#9DCC00", "#C20088", "#003380", "#FFA405",
    "#FFA8BB", "#426600", "#FF0010", "#5EF1F2", "#00998F",
    "#E0FF66", "#740AFF", "#990000", "#FFFF80", "#FFFF00",
    "#FF5005"
    )
```
```
colors <- c(
    '#000000', '#010067', '#d5ff00', '#ff0056', '#9e008e',
    '#0e4ca1', '#ffe502', '#005f39', '#00ff00', '#95003a',
    '#ff937e', '#a42400', '#001544', '#91d0cb', '#620e00',
    '#6b6882', '#0000ff', '#007db5', '#6a826c', '#00ae7e',
    '#c28c9f', '#be9970', '#008f9c', '#5fad4e', '#ff0000',
    '#ff00f6', '#ff029d', '#683d3b', '#ff74a3', '#968ae8',
    '#98ff52', '#a75740', '#01fffe', '#ffeee8', '#fe8900',
    '#bdc6ff', '#01d0ff', '#bb8800', '#7544b1', '#a5ffd2',
    '#ffa6fe', '#774d00', '#7a4782', '#263400', '#004754',
    '#43002c', '#b500ff', '#ffb167', '#ffdb66', '#90fb92',
    '#7e2dd2', '#bdd393', '#e56ffe', '#deff74', '#00ff78',
    '#009bff', '#006401', '#0076ff', '#85a900', '#00b917',
    '#788231', '#00ffc6', '#ff6e41', '#e85ebe'
    )
```


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
#### matrix conversion
`m_com = as.matrix(com)`

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

### plotly
`plot_ly(x = data$V6, y = data$V5, text=data$V1,
marker=list(opacity=0.2), type='scatter', mode='marker')`

remove the marker options to not have opacity. This opacity setting makes
overlapping points darker

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

<br />

##### optional: prepping the fit
```
mds.fit = envfit(nmds3, com, permutations = 9999, na.rm = TRUE)
mds_coords = as.data.frame(scores(mds.fit, 'vectors') * ordiArrowMul(mds.fit))
```

<br/>

#### plotting (3D)
```
plot <- plot_ly( x = data.scores$NMDS1, y = data.scores$NMDS2, z = data.scores$NMDS3, color = data.scores$sphy, colors = colors )
```

<br />

#### with the fit (3D)
```
for ( row in 1:nrow(mds_coords) ) {
    t_row <- mds_coords[row,]
    t_row[2,] <- 0
    plot <- plot %>% add_trace(
        t_row, x = t_row$NMDS1, y = t_row$NMDS2, z = t_row$NMDS3,
        opacity = 1, type = 'scatter3d', mode = 'lines', line = list(
            width = 3, color = 'black'
            ),
        showlegend = FALSE
    )
}
```
