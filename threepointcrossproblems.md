---
title: "three-point-cross-problems"
author: "em"
date: "November 6, 2019"
output:
  html_document:
    keep_md: yes
---



Finding 3-point cross problems can be a huge pain, so here I'll work out how to make them myself.

Imagine we have a genetic map where there are three genes (A, B, C). The map distance between A and B is X, the map distance between B and C is Y.



```r
myX=10
myY=20
geneNames = c('A','B','C')

linkageMap <- function(geneNames = c('A','B','C'), myX = 10, myY = 10, myLabels = c(myX, myY)){
myZ = myX + myY
plot(0,type='n',axes=FALSE,ann=FALSE, xlim = c(-0.3,1.3), ylim = c(0,1))
lines(c(0.05,myX/myZ-0.05), c(0.5,0.5), lwd=2)
lines(c(myX/myZ+0.05, 0.95), c(0.5,0.5), lwd=2)
text(c(0,myX/myZ,1), c(0.5,0.5,0.5), labels = geneNames, cex=2)
text(c(0.5*myX/myZ,  myX/myZ + 0.5*myY/myZ) ,c(0.6,0.6),labels=myLabels, cex=2)
}

linkageMap(myX=15, myY = 20, myLabels = c('X','Y'))
```

![](threepointcrossproblems_files/figure-html/makefigure-1.png)<!-- -->


We do a three point test cross and get $N$ progeny.

For now I'll assume that one parent is (A B C/ a b c) and the other parent is (a b c/a b c) but I can revisit this assumption later.

The following math will assume that there's no interference (can revisit this later).
Let $x$ and $y$ be the recombination frequencies, so that $x = \frac{X}{100}$ and $y = \frac{Y}{100}$. 
We can work out the proportions of all the different gametes contributed by the heterozygous parent using the following formulas.

First, the double crossovers. The probability of getting crossovers in both intervals is $x\cdot y$.

$$P(AbC) =  N\cdot (x\cdot y)/2 $$
$$P(aBc) =  N \cdot (x\cdot y)/2 $$
Next, for the single crossovers, we calcuate the probablity of crossovers occuring in one interval but not in both intervals, which is equibalent to $x - xy$ for crossovers between A and B and $y - xy$ for crossovers between B and C. 

$$P(Abc) =  N\cdot (x - xy)/2 $$
$$P(aBC) =  N\cdot (x - xy)/2 $$
and

$$P(ABc) =  N\cdot (y - xy)/2 $$
$$P(abC) =  N\cdot (y - xy)/2 $$
Finally, we figure out the frequency of non-recombinant gametes. This is a bit tricky because it's not just equivalent to $1 - x - y$ because we need to account for double crossovers. Instead, the frequency of non-recombinant gametes is $1 - x - y + xy$

$$P(ABC) =  N\cdot (1 - x - y + xy)/2 $$
$$P(abc) =  N\cdot (1 - x - y + xy)/2 $$


Now to code this up in R.

```r
calculateProgeny <- function(myX = 10, myY = 20, myN = 1000){
myx = myX/100
myy = myY/100

myDoubleCrossovers = myN * myx * myy 
myABRecombinants = myN * (myx - myx*myy)
myBCRecombinants = myN * (myy - myx*myy)
myNonrecombinants = myN* (1 - myx - myy + myx*myy)

myNumbers =c( myNonrecombinants/2, myNonrecombinants/2, 
              myABRecombinants/2, myABRecombinants/2, 
              myBCRecombinants/2, myBCRecombinants/2,
              myDoubleCrossovers/2, myDoubleCrossovers/2)

myProgeny = c('ABC','abc','aBC','Abc','abC','ABc','aBc','AbC')
myOutput = data.frame(progeny = myProgeny, numbers = myNumbers, stringsAsFactors = F)

library(knitr)
kable(myOutput)
}
```

Now that this is coded up we can try this for a couple different maps.

For example,


```r
linkageMap(myX = 5, myY = 3)
```

![](threepointcrossproblems_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
calculateProgeny(myX=5, myY=4, myN = 500)
```



|progeny | numbers|
|:-------|-------:|
|ABC     |   228.0|
|abc     |   228.0|
|aBC     |    12.0|
|Abc     |    12.0|
|abC     |     9.5|
|ABc     |     9.5|
|aBc     |     0.5|
|AbC     |     0.5|

```r
linkageMap(myX = 15, myY = 20)
```

![](threepointcrossproblems_files/figure-html/unnamed-chunk-2-2.png)<!-- -->

```r
calculateProgeny(myX=15, myY=20, myN = 800)
```



|progeny | numbers|
|:-------|-------:|
|ABC     |     272|
|abc     |     272|
|aBC     |      48|
|Abc     |      48|
|abC     |      68|
|ABc     |      68|
|aBc     |      12|
|AbC     |      12|

```r
linkageMap(myX = 25, myY = 7)
```

![](threepointcrossproblems_files/figure-html/unnamed-chunk-2-3.png)<!-- -->

```r
calculateProgeny(myX=25, myY=7, myN = 800)
```



|progeny | numbers|
|:-------|-------:|
|ABC     |     279|
|abc     |     279|
|aBC     |      93|
|Abc     |      93|
|abC     |      21|
|ABc     |      21|
|aBc     |       7|
|AbC     |       7|
