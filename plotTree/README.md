Plotting trees with data using R and Python
====
[R code - plotTree.R](#r-code)

[Python code (using ete2) - plotTree.py](#python-code)

This is rough code that we use sometimes for making figures in my lab (http://holtlab.net). 

There will definitely be bugs and there's lots more features one could add, and probably many ways these features could be implemented differently/better.

So basically, I am putting this up here to share the love and save other people's time messing around with R, Python and ETE2, and so that others can use the code as the basis for their own functions or to learn how to do various handy things with these packages. If you find it useful and expand on it - please share!

Kat Holt - @DrKatHolt - http://holtlab.net

R code
==

# plotTree.R

This is R code for plotting a phylogenetic tree and annotating the leaves with various information, including: 
- colouring tips according to some variable (provided in infoFile; CSV format with column 1 = tip names)
- printing columns of text next to the leaves (provided in infoFile; CSV format with column 1 = tip names)
- printing heatmaps of data (provided in heatmapData; numerical data in CSV format with column 1 = tip names)
- printing horizontal bar graphs next to the tips (provided in barData; numerical data in CSV format with column 1 = tip names)
- printing the location of SNPs (snpFile; allele data in CSV format with row 1 = tip names; SNPs relative to reference, either column 1 or a specified strain)
- printing the location of genome blocks (blockFile; tab delimited file with col 1 = tip name, col 2 = start, col 3 = stop)

There are also options to:
- cluster the heatmap data using any method available in hclust
- perform ancestral discrete trait reconstruction using ace and plot the results as pie graphs on each node of the tree

# Overview

1) Prepare your tree file (to be passed to the function via tree="tree.nwk")
--
- newick format
- no hashes in the strain names (trees can't be read into R if they have hashes)
- alternatively, this can be an R object of the class 'phylo' (can convert hclust object to this format using as.phylo())

2) Prepare your data files
--
- you can provide any or all of strain info, data to be plotted as a heatmap, data to be plotted as a bar chart, snp allele table in the order: tree | info | heatmap | barplot | SNPS/blocks
- CSV format, one row per strain (EXCEPT SNP allele table or blocks files, which contain coordinates of SNPs and blocks, see above)
- column 1 contains strain names, precisely matching those in the tree file
- row 1 contains variable names
- alternatively, these can be R objects of class 'matrix' or 'data.frame'

3) Optional input data files (any combination can be provided, but they will always be plotted in this order across the page, with the tree on the left):
--
(i) Strain info / metadata (to be passed to the function via infoFile="info.csv")
- the values in the columns will be printed (in columns) next to the tree
- optionally, if you have lots of columns and only want to print some of them, you can specify the names of the columns to print using infoCols=c("variable1","variable2"); otherwise all columns will be printed
- optionally, if you want to colour the tree tips according to the value of one of the data columns, specify the name of the variable via colourNodesBy="variable"; you can also perform ancestral trait reconstruction on this variable and plot the results as pie graphs, to turn this on use ancestral.reconstruction=T

(ii) Numeric values to plot as a heatmap (to be passed to the function via heatmapData="data.csv")

(iii) One column of numeric values to plot as a barplot (to be passed to the function via barData="bar.csv")

(iv) SNP allele table (to be passed to the function via snpFile="alleles.csv")
 will plotted to indicate the position of SNPs in each strain, where SNPs are defined as differences COMPARED TO THE ALLELES IN COLUMN 1. So, your alleles in column one should be the inferred ancestral alleles (e.g. those of an outgroup).
- note you need to specify the total size of the genome in base pairs, to set up the appropriate X-axis; set using genome_size
- unknown alleles or gaps should not be plotted as SNPs compared to the ancestral; the gap character is assumed to be '?', but often this will be '-' if your data comes direct from the mapping pipe, so you will need to change to gapChar="-"

(v) Blocks file (to be passed to the function via blockFile="blocksByStrain.txt")
- note you need to specify the total size of the genome in base pairs, to set up the appropriate X-axis; set using genome_size

Tree plotting function
--
    p <- plotTree(tree="tree.nwk",heatmapData="data.csv",infoFile="info.csv",barData="bar.csv",snpFile="alleles.csv", blockFile="blocksByStrain.txt")


Optionally, output to PDF:
--
    outputPDF="out.pdf"

(specify width in inches via w=X, specify height in inches via h=X)

OR output to PNG:

    outputPNG="out.png"

(specify width in pixels via w=X, specify height in pixels via h=X)

Spacing options
--
You can provide any or all of strain info, data to be plotted as a heatmap, data to be plotted as a bar chart, SNPs and/or blocks. 

The order will be:

[ tree | info | heatmap | barplot | SNPs/blocks]

• Relative widths of the components can be changed in the function; by default they are:


left & right spacing framing the whole page: edgeWidth = 1

tree plotting space: treeWidth = 10

info printing space: infoWidth = 10

heatmap printing space: dataWidth = 30

barplot plotting space: barDataWidth = 10

SNP/blocks plotting space: blockPlotWidth = 10

• Relative heights of the components can be changed in the function; by default they are:

height of plotting spaces: mainHeight = 100

top & bottom spacing: labelHeight = 10    

  - if heatmap provided, this will be the height of the area in which the column names are printed above the heatmap; otherwise the top edge height will be taken from edgeWidth

  - if barplot provided, this will be the height of the area in which the x-axis is printed below the barplot; otherwise the bottom edge height will be taken from edgeWidth



Tree options
--
(see ?plot.phylo in R for more info)

• tip.labels = T     turns on printing strain names at the tips

• tipLabelSize = 1     change the size of printed strain names (only relevant if tip.labels=T)

• offset=0     change the spacing between the end of the tip and the printed strain name (only relevant if tip.labels=T)

• tip.colour.cex=0.5    change the size of the coloured circles at the tips (only relevant if infoFile is provided AND colourNodesBy is specified)

• tipColours = c("blue","red","black")    specify colours to use for colouring nodes (otherwise will use rainbow(n)). RColourBrewer palettes are a good option to help with colour selection

• lwd=1.5    change the line width of tree branches

• edge.color="black"    change the colour of the tree branches

• axis=F,axisPos=3     add and position an axis for branch lengths



Info options
--
• colourNodesBy = "column name"    colour the nodes according to the discrete values in this column. additional options:

- legend=T, legend.pos="bottomleft"    plot legend of node colour values, specify location (possible values: "topleft","topright","bottomleft" or "bottomright")

- ancestral.reconstruction=T     reconstruct ancestral states for this discrete variable, results will be returned as $mat and plotted as pie graphs on the tree

• infoCex=0.8     Change the size of the printed text


Heatmap options
--
• heatmap.colours=

- if not specified, uses white -> black

- colorRampPalette is a good option, eg:

    heatmap.colours=colorRampPalette(c("white","yellow","blue"),space="rgb")(100)

- note the legend/scale will be plotted above the tree

• colLabelCex=0.8       change the size of the column labels

• cluster     Cluster matrix columns?  (Default is no clustering.)

- Set cluster=T to use default hclust clustering method ("ward.D"), or specify a different method to pass to hclust (see ?hclust for options).

- Alternatively, if you have a square matrix (i.e. strain x strain) and you want to order columns the same as rows to keep it square, set cluster="square"


Barplot options
--
• barDataCol=2     Colour for the barplot (can be numeric, 1=black, 2=red, etc; or text, "red", "black", etc)


SNP plot options
--
• genome_size     Sets the length of the x-axis that represents the length of the genome. This is REQUIRED when plotting SNPs/blocks.

• gapChar="-"     Character used to indicate gaps/unknown alleles in the SNP file (will not be counted as SNPs).

• snp_colour     Sets the colour of the lines indicating SNPs (default is red)


Block plot options
--
• genome_size     Sets the length of the x-axis that represents the length of the genome. This is REQUIRED when plotting SNPs/blocks.

• block_colour     Sets the colour of the lines indicating blocks (default is black). Blocks are drawn after SNPs, so may obscure SNPs.

• blwd     Sets the height of the lines indicating blocks (default is 5).

Ancestral trait reconstruction
--
To perform ancestral discrete trait reconstruction using ace, and plot the results as pie graphs on each node of the tree:     

(i) specify the variable in the infoFile that you want to analyse: colourNodesBy="Variable_name"

(ii) set ancestral.reconstruction = T

(iii) to change the size of the pie graphs, change pie.cex (default value is 0.5)

Outputs
--
Primary output is the rendered tree figure (in the R drawing device or in a PDF/PNG file if specified)
The plotTree() function also returns an R object with the following:

$info: infoFile input file, re-ordered as per tree

$anc: result of ancestral discrete trait reconstruction using ace

$mat: heatmap data file, with rows re-ordered as per tree and columns re-ordered as per clustering (if cluster=T)

$strain_order: order of leaves in the tree


# Examples

Data (trees and tables) used in this example are available in the subdirectory /tree_example_april2015

Basic strain info
---
Plot tree, colour tips by city of isolation, specify colours for each city manually, print strain details as table next to tree.

v <- plotTree(tree="tree.nwk",ancestral.reconstruction=F,tip.colour.cex=1,cluster=T,tipColours=c("black","purple2","skyblue2","grey"),lwd=1,infoFile="info.csv",colourNodesBy="location",treeWidth=10,infoWidth=10,infoCols=c("name","location","year"))

![](tree_example_april2015/info.png?raw=true)

Pan genome heatmap
---
Plot tree, colour tips by location (as above), cluster a gene content matrix and plot as heatmap next to the tree (white = 0% coverage of gene, black = 100% coverage of the gene).

v <- plotTree(tree="tree.nwk",heatmapData="pan.csv",ancestral.reconstruction=F,tip.colour.cex=1,cluster=T,tipColours=c("black","purple2","skyblue2","grey"),lwd=1,infoFile="info.csv",colourNodesBy="location",treeWidth=5,dataWidth=20,infoCols=NA)

![](tree_example_april2015/pan.png?raw=true)

Curated genes, coloured
---
Plot tree, colour tips by location (as above), plot curated resistance gene information next to the tree as a heatmap... 

Here the gene information in the heatmapData file is coded so that 0 represents absence, and different numbers are used to indicate presence of each gene/variant (e.g. in the gyrA column, one mutation is coded as 2 and the other is coded as 4).

We then specify which colour to use for each number, using heatmap.colours... here 0 (ie absent) is white; 2 (ie gyrA mutant 1) is "seagreen3"; 4 (ie gyrA mutant 2) is "darkgreen", etc etc.

heatmap.colours=c("white","grey","seagreen3","darkgreen","green","brown","tan","red","orange","pink","magenta","purple","blue","skyblue3","blue","skyblue2")

v <- plotTree(tree="tree.nwk",heatmapData="res_genes.csv",ancestral.reconstruction=F,tip.colour.cex=1,cluster=F,heatmap.colours=c("white","grey","seagreen3","darkgreen","green","brown","tan","red","orange","pink","magenta","purple","blue","skyblue3","blue","skyblue2"),tipColours=c("black","purple2","skyblue2","grey"),lwd=1,infoFile="info.csv",colourNodesBy="location",treeWidth=10,dataWidth=10,infoCols=c("name","year"),infoWidth=8)

![](tree_example_april2015/res_genes.png?raw=true)


Python code
==

# plotTree.py

This is a Python script, that uses the Python package ETE2 (see http://ete.cgenomics.org/) which in turn requires pyqt4 and numpy. If you have BioPython then you will already have numpy. To install the rest, follow these steps:

1. Download sip from http://www.riverbankcomputing.com/software/sip/download

2. Unpack it (tar -xzvf sip.tar.gz) and cd into the directory (cd sip...)

3. Run these commands:

  python configure.py -d /Library/Python/2.7/site-packages --arch x86_64
  
  make
  
  sudo make install

4. Download the Qt4 binary from http://qt.nokia.com/downloads/sdk-mac-os-cpp

5. download PyQt4 from http://pyqt.sourceforge.net/Docs/PyQt4/installation.html

6. unpack it and cd into the directory

7. Run these commands:

  python configure.py -q /usr/bin/qmake-4.8 -d /Library/Python/2.7/site-packages/ --use-arch x86_64
  
  make
  
  sudo make install

8. Now you can install ETE2

  sudo easy_install -U ete2

To use ETE2 on your own, see http://ete.cgenomics.org/

Running on contagion or merri

The script will work on a remote server, but ONLY IF you add "-Y" to the ssh login command. Eg:

  ssh -Y you@server.com


plotTree.py - Options
==

# Required inputs

--output tree.pdf

- Name of output file
- must end in .pdf or .png
- the file type for the ouput figure is taken from this extension

--tree tree.nwk

- Tree file (newick format)

--info info.csv

- Data file in CSV format; column 1 matches leaf names, other columns contain data for labelling or plotting.

Labelling and colour options (text or categorical values)
==
# (i) Print values directly as columns of text
--labels LABELS [LABELS ...]

- labels to print as text columns next to the tree (must match column headers in the info file)

--tags

- Switch on colour labelling of backgrounds to indicate values

--padding PADDING     

- padding between label columns (pixels, default 20)

# (ii) Colour the leaf nodes
--colour_nodes_by COLOUR_NODES_BY

- label to use for colouring nodes

--node_size NODE_SIZE

-size for node shapes (default 10)

# (iii) Use colour blocks to represent column values

--colour_tags COLOUR_TAGS [COLOUR_TAGS ...]

- labels to use to tag each element by colour code

# (iv) Specify colours directly (otherwise expanded ColorBrewer Set1 is used)

--colour_dict COLOUR_DICT

- manually specify dictionary of values -> colours

Data matrix options (numerical data)
==

--data DATA           

- Data matrix (tab delmited; header row starts with "#Names col1 col2 etc...", column 1 matches leaf names, other columns should be numerical values for plotting)

--data_type DATA_TYPE

- Type of data plot ([heatmap], line_profiles, bars, cbars)
- See ete2 docs for options

--data_width DATA_WIDTH

Total width of data plot for each strain (mm, default 200)

--data_height DATA_HEIGHT


- Total height of data plot for each strain (mm, default 20)

--mindata MINDATA     Minimum data value for plotting scale (-1)

--maxdata MAXDATA     Maximum data value for plotting scale (1)

--centervalue CENTERVALUE

- Central data value for plotting scale (0)

Tree formatting options
==
--midpoint            Midpoint root the tree

--outgroup OUTGROUP   Outgroup to root the tree

--no_ladderize        Switch off ladderizing

--show_leaf_names     Print leaf names as well as labels

--branch_support      Print branch supports

--no_guiding_lines    Turn off linking nodes to data with guiding lines

--fan                 Plot tree as fan dendrogram

--length_scale LENGTH_SCALE      scale (pixels per branch length unit)

--branch_padding BRANCH_PADDING   branch (pixels between each branch, ie vertical padding)

Branch supports
==
--branch_support_print   Print branch supports

--branch_support_colour  Colour branch leading to node by branch supports (scale is 0=red -> 100=black)

--branch_support_cutoff BRANCH_SUPPORT_CUTOFF  

- Colour branches with support lower than this value (<cutoff = red; >= cutoff = black)

Colour branches or clade backgrounds by value of descendant leaves
==
--colour_branches_by COLOUR_BRANCHES_BY  variable to use for colouring branches

--colour_backgrounds_by COLOUR_BACKGROUNDS_BY  variable to use for colouring clade backgrounds

Plotting options
==
--title TITLE         Title for plot

--width WIDTH         width of output image pile (mm, default 200)

--interactive         Switch on interactive view (after printing tree to file)


plotTree script - examples
==
COMING SOON
