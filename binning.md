# Binning data in python

#### specify range to bin by
`indices = list( range( 0, 50, 2 ) )`

#### create list of tuples to bin
```
bin_tups = []
count = 0
for index in indices:
	if count != 0:
		bin_tups.append( (indices[count - 1], index) )
	count += 1
```

#### use pandas to bin
```
bins = pd.IntervalIndex.from_tuples(bin_tups)
cat_obj = pd.cut( $DATA, bins )
```

#### convert to pandas series
`series = pd.value_counts( cat_obj )`
