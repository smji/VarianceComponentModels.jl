
# Heritability Analysis

This note demonstrates the workflow for a typical heritability analysis in genetics, using a sample data set `cg10k` with **6,670** individuals and **630,860** SNPs. Person IDs and phenotype names are masked for privacy. Here `cg10k.bed`, `cg10k.bim`, and `cg10k.fam` is a set of Plink files in binary format. `cg10k_traits.txt` contains the phenotype data of 6,670 individuals.


```julia
;ls cg10k*.*
```

    cg10k.bed
    cg10k.bim
    cg10k.fam
    cg10k.ipynb
    cg10k.jld
    cg10k_traits.txt


Machine information:


```julia
versioninfo()
```

    Julia Version 0.4.6
    Commit 2e358ce (2016-06-19 17:16 UTC)
    Platform Info:
      System: Darwin (x86_64-apple-darwin13.4.0)
      CPU: Intel(R) Core(TM) i7-4790K CPU @ 4.00GHz
      WORD_SIZE: 64
      BLAS: libopenblas (USE64BITINT DYNAMIC_ARCH NO_AFFINITY Haswell)
      LAPACK: libopenblas64_
      LIBM: libopenlibm
      LLVM: libLLVM-3.3


## Read in binary SNP data

We will use the [`SnpArrays`](https://github.com/Hua-Zhou/SnpArrays.jl) package to read in binary SNP data and compute the empirical kinship matrix. Issue `Pkg.clone("git@github.com:Hua-Zhou/SnpArrays.jl.git")` within `Julia` to install the `SnpArrays` package.


```julia
#Pkg.clone("git@github.com:OpenMendel/SnpArrays.jl.git")
using SnpArrays
```


```julia
# read in genotype data from Plink binary file (~50 secs on my laptop)
@time cg10k = SnpArray("cg10k")
```

     33.409278 seconds (4.58 k allocations: 1003.520 MB, 0.07% gc time)





    6670x630860 SnpArrays.SnpArray{2}:
     (false,true)   (false,true)   (true,true)   …  (true,true)    (true,true) 
     (true,true)    (true,true)    (false,true)     (false,true)   (true,false)
     (true,true)    (true,true)    (false,true)     (true,true)    (true,true) 
     (true,true)    (true,true)    (true,true)      (false,true)   (true,true) 
     (true,true)    (true,true)    (true,true)      (true,true)    (false,true)
     (false,true)   (false,true)   (true,true)   …  (true,true)    (true,true) 
     (false,false)  (false,false)  (true,true)      (true,true)    (true,true) 
     (true,true)    (true,true)    (true,true)      (true,true)    (false,true)
     (true,true)    (true,true)    (false,true)     (true,true)    (true,true) 
     (true,true)    (true,true)    (true,true)      (false,true)   (true,true) 
     (true,true)    (true,true)    (false,true)  …  (true,true)    (true,true) 
     (false,true)   (false,true)   (true,true)      (true,true)    (false,true)
     (true,true)    (true,true)    (true,true)      (true,true)    (false,true)
     ⋮                                           ⋱                             
     (false,true)   (false,true)   (true,true)      (false,true)   (false,true)
     (false,true)   (false,true)   (false,true)     (false,true)   (true,true) 
     (true,true)    (true,true)    (false,true)  …  (false,true)   (true,true) 
     (false,true)   (false,true)   (true,true)      (true,true)    (false,true)
     (true,true)    (true,true)    (true,true)      (false,true)   (true,true) 
     (true,true)    (true,true)    (true,false)     (false,false)  (false,true)
     (true,true)    (true,true)    (true,true)      (true,true)    (false,true)
     (true,true)    (true,true)    (false,true)  …  (true,true)    (true,true) 
     (true,true)    (true,true)    (true,true)      (false,true)   (true,true) 
     (true,true)    (true,true)    (true,true)      (true,true)    (false,true)
     (false,true)   (false,true)   (true,true)      (true,true)    (true,true) 
     (true,true)    (true,true)    (true,true)      (true,true)    (true,true) 



## Summary statistics of SNP data


```julia
people, snps = size(cg10k)
```




    (6670,630860)




```julia
# summary statistics (~50 secs on my laptop)
@time maf, minor_allele, missings_by_snp, missings_by_person = summarize(cg10k)
```

     28.517763 seconds (20 allocations: 9.753 MB)





    ([0.169916,0.17099,0.114026,0.268694,0.219265,0.23935,0.190612,0.20201,0.0271609,0.299714  …  0.207996,0.448012,0.295465,0.142654,0.170915,0.281428,0.0611354,0.0524738,0.13931,0.132413],Bool[true,true,true,true,true,true,true,true,true,true  …  true,true,true,true,true,true,true,true,true,true],[2,0,132,77,0,27,2,2,6,27  …  4,5,11,0,0,4,29,0,5,43],[364,1087,1846,1724,1540,248,215,273,273,224  …  1353,692,901,1039,685,304,223,1578,2059,1029])




```julia
# 5 number summary and average MAF (minor allele frequencies)
quantile(maf, [0.0 .25 .5 .75 1.0]), mean(maf)
```




    (
    1x5 Array{Float64,2}:
     0.00841726  0.124063  0.236953  0.364253  0.5,
    
    0.24536516625042462)




```julia
using Plots
pyplot()
#gr()

histogram(maf, xlab = "Minor Allele Frequency (MAF)", label = "MAF")
```




<img src="data:image/png;base64,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5ocfflCPHj30+eefa8GCBVqzZo3+8Ic/6JZbbtHx48cvu83CwkKdO3dOv/32m3777TedO3fOpf2ee+5R37599c477yg+Pr7M23v88cf16quvavr06Vq9erXOnTunxx57rMynNy+e7+DBg+rWrZvy8vK0dOlSrVixQsePH9ett96qX3/91WW5BQsW6Ntvv9Wbb76pp59+WitWrNAzzzzjbD958qRuvvlmbd26Vc8++6w2btyohx9+2Hlkady4ccrNzdX777/vst4lS5bo5ptvVsuWLUus+Z///Ke6detWYvuIESNcAnNaWpqGDBmi2rVrFzt/eT5XF/fhxX+kXHDzzTdr+/btOnv2bIn1AVWWBaDcZsyYYV111VWWZVnWvffea8XHx1uWZVlPPfWU1b17d8uyLKt9+/bW6NGjXZapW7eu8/WHH35o2Ww2a/r06S7rbt68uTV27Fjn66ioKOc6L/j73/9u2Ww2a//+/S7reuCBB8pUf0FBgRUYGOicv7Cw0GrSpIk1atQol/nuu+8+q23bts7XKSkpls1ms06ePGlZlmV98MEHls1ms7Zs2eKyXFxcnHXjjTeWuJ6nn37aCgwMtI4fP+5SU0hIiDVt2rRSaw8JCbFsNpvLT40aNVzqe/75512WKcv2Tp48aQUEBFgzZsxwWbZ3796WzWazDh06ZFnWf9/rPXv2uMx31113WdHR0c7Xo0aNssLCwqyCggLntOPHj1tXXXWV9corrzin2Ww266abbnJZV0JCghUWFuZ8/cQTT1i1a9d21lCcnj17WrGxsc7XJ06csGrVqmW9/vrrJS7z22+/Wf7+/tbLL79cpM1ms1kvvPCC5XA4LJvNZh08eND6+eefrTp16ljvv/++tXbtWpf3xbLK/rmaMWNGkT602WzW8uXLXebbvn27ZbPZrL1795a4D0BVxREs4ApZvx+hiouL0/r163XmzBmlp6drxIgR5VrPn/70J5fXrVq10nfffSdJ+s9//qPPPvtMd999t8s8F07/ffTRRy7TSzptc6lNmzYpLy9PQ4cOlXT+yMvgwYO1du3acl2z9N577ykoKEh9+vRxORLRt29f7du3r8Trrt577z316dNHgYGBzmXsdrt69eqlXbt2lbpNm82m2NhY7d692/mzY8cOl3kufR/Ksr39+/frzJkzGjx4sMuyQ4YMKfP7cek2Y2JiZLfbndusV6+eOnToUGQfb7vtNpfXF38GpPNHmW699dZSL+QfN26c1q9fr7y8PEnnjyTVrFlTcXFxJS5z8uRJnTt3rsjF7RcLCwtTp06dtGLFCq1bt05XXXWVbr311mLnLc/nKiAgwKUPd+/erdtvv91lnvr160uSjh07VmJ9QFXFXYSAm/r166caNWooKSlJ2dnZzvBT1tNK9erVc3ldo0YN/fTTT5KkvLw8WZal6667zmWea665RrVq1SoyXMKl85Vk+fLlqlevntq3b+/8Qr7tttv0yiuvaMOGDZe9fuuCEydO6NSpU6pRo0aRNpvNpiNHjqhJkybFLrdjx45ilwsLC7vsdhs0aKCOHTuW2H7p+1CW7R05ckSS1LBhw1LXVZqLA+WJEyc0f/58zZ8/v8h8l55eu/QzULNmTZdr7E6dOqV27dqVuu1hw4Zp0qRJSk1N1YMPPqiUlBTdfffd+sMf/lDm+ksyYsQILVmyRCEhIYqNjS3xs12ez5Xdbi+1D6X/vp+m7kAFKhMBC3BTjRo1NHToUP31r39V37591aBBA2Prrlevnmw2W5G/4H/88UcVFBQUOfJQli+in3/+Wf/3f/+nM2fOFFvr8uXLyxywgoKC1KBBA23atKnY9pLei/r16ysiIsLlOqMLatWqVaZtl+bS96Es22vcuLGk80dLLvy/JB09etRl/gvh6NLrqH744Qf5+fm5bHPAgAF64IEHimyzvOOh1a9fX4cPHy51ntq1a+vee+9VSkqKunfvrs8++0wLFiy47Hr9/Px08uTJUueLjY3Vo48+qgMHDigpKanYeUx+ri648AfEpaEX8AYELMCAsWPH6vjx4xo3bpzR9datW1cdOnTQ6tWrNWnSJOf0VatWSZJ69OhR7nVeOF3z2muvFblzLCUlRStWrFBeXl6RoyrFue222zR37lzVqFGjXAOJ9u3bV8uWLVPLli2dYx9VpLJsLzIyUgEBAXr77bfVvn175/Q1a9a4zNesWTNJ529ouOmmmySdP1q1d+9edenSxWWb+/fvV4cOHWS3u3c1Rt++fTVv3jzl5ubq+uuvL3G+cePGKTk5WY888ogiIiLUvXv3Utfr5+enyMhIffHFF6XO17RpUz388MM6ceKEc58vZfJzdcEXX3yh2rVrq02bNmVeBqgqCFiAAV26dCkyXo9lWVc89tPFy82cOVODBg1SfHy87r33Xh04cEBPPvmk7r777iv64lm+fLlCQ0OLDYOBgYFaunSpVq1apfHjx192XX379lVMTIz69++vadOmKTIyUqdPn9aXX36pb7/9VosWLSp2uUceeUTLly9X7969NWnSJF1//fU6fvy4duzYoaZNm2ry5MklbvNK3tOybC8oKEgTJkzQc889p4CAAEVFRSktLU0HDx50OSLWrFkzde3aVbNmzdI111wjPz8/zZkzR/Xq1XOpbdasWerSpYv69eun8ePHq2HDhvr++++VkZGhXr16lXpt1KUefvhhvfnmm+rVq5eSkpLUvHlzHTx4UA6HQ88995xzvnbt2qlLly7aunWry/TS3HrrraXe8XnBCy+8UGq7yc/VBR9//LF69OhR7KldoKrjInfgCthstsuejrt0nuKWKW4dl84XExOj1atXa//+/Ro0aJCef/553X///UUeI1KW04PHjh3TBx98oPj4+GLbIyMj1aFDB61YsaLMNb/11luaMGGCXnnlFd1xxx0aO3astmzZoj59+pS4T0FBQfrkk0/UoUMHTZ8+Xf369dMjjzyinJycEo+QlHU/i2sv6/aee+45TZgwQc8//7xiY2Nlt9v13HPPFQl1y5cvV1hYmBISEjRt2jQ9/PDD6ty5s8u2W7RooZ07d6p+/fp64IEH1L9/fz3++OP65ZdfXI6QlWU/goKC9NFHH6lHjx6aNm2a7rzzTr344ovFHs0aNGiQ/Pz8dN999112G5I0evRoffnllzpw4ECZ5i+uxvJ+ri5etiS//vqrNmzY4PK4KcCb2Kwr/RMbAKqBdevWaciQIcrOzvaKx/H06tVLgYGBWr9+fZmXuf3229W2bdtiB7n1lBUrVuiJJ56Qw+HgCBa8EqcIAcAH7N69W9u2bdP27dvLPWDsc889p969e+vxxx8vdciGylJYWKj//d//1ezZswlX8FoELAC4DG8YJuDGG29UvXr19PTTT+uWW24p17IXD6tQFdjtdn355ZeeLgNwy/8DWvh5J23zHFkAAAAASUVORK5CYII=" />




```julia
# proportion of missing genotypes
sum(missings_by_snp) / length(cg10k)
```




    0.0013128198764010824




```julia
# proportion of rare SNPs with maf < 0.05
countnz(maf .< 0.05) / length(maf)
```




    0.07228069619249913



## Empirical kinship matrix

We estimate empirical kinship based on all SNPs by the genetic relation matrix (GRM). Missing genotypes are imputed on the fly by drawing according to the minor allele frequencies.


```julia
# GRM using all SNPs (~10 mins on my laptop)
srand(123)
@time Φgrm = grm(cg10k; method = :GRM)
```

    343.981385 seconds (4.21 G allocations: 64.468 GB, 2.25% gc time)





    6670x6670 Array{Float64,2}:
      0.502916      0.00329978   -0.000116213  …  -6.46286e-5   -0.00281229 
      0.00329978    0.49892      -0.00201992       0.000909871   0.00345573 
     -0.000116213  -0.00201992    0.493632         0.000294565  -0.000349854
      0.000933977  -0.00320391   -0.0018611       -0.00241682   -0.00127078 
     -7.75429e-5   -0.0036075     0.00181442       0.00213976   -0.00158382 
      0.00200371    0.000577386   0.0025455    …   0.000943753  -1.82994e-6 
      0.000558503   0.00241421   -0.0018782        0.001217     -0.00123924 
     -0.000659495   0.00319987   -0.00101496       0.00353646   -0.00024093 
     -0.00102619   -0.00120448   -0.00055462       0.00175586    0.00181899 
     -0.00136838    0.00211996    0.000119128     -0.00147305   -0.00105239 
     -0.00206144    0.000148818  -0.000475177  …  -0.000265522  -0.00106123 
      0.000951016   0.00167042    0.00183545      -0.000703658  -0.00313334 
      0.000330442  -0.000904147   0.00301478       0.000754772  -0.00127413 
      ⋮                                        ⋱                            
      0.00301137    0.00116042    0.00100426       6.67254e-6    0.00307069 
     -0.00214008    0.00270925   -0.00185054      -0.00109935    0.00366816 
      0.000546739  -0.00242646   -0.00305264   …  -0.000629014   0.00210779 
     -0.00422553   -0.0020713    -0.00109052      -0.000705804  -0.000508055
     -0.00318405   -0.00075385    0.00312377       0.00052883   -3.60969e-5 
      0.000430196  -0.00197163    0.00268545      -0.00633175   -0.00520337 
      0.00221429    0.000849792  -0.00101111      -0.000943129  -0.000624419
     -0.00229025   -0.000130598   0.000101853  …   0.000840136  -0.00230224 
     -0.00202917    0.00233007   -0.00131006       0.00197798   -0.000513771
     -0.000964907  -0.000872326  -7.06722e-5       0.00124702   -0.00295844 
     -6.46286e-5    0.000909871   0.000294565      0.500983      0.000525615
     -0.00281229    0.00345573   -0.000349854      0.000525615   0.500792   



## Phenotypes

Read in the phenotype data and compute descriptive statistics.


```julia
using DataFrames

cg10k_trait = readtable("cg10k_traits.txt"; 
    separator = ' ',
    names = [:FID; :IID; :Trait1; :Trait2; :Trait3; :Trait4; :Trait5; :Trait6; 
             :Trait7; :Trait8; :Trait9; :Trait10; :Trait11; :Trait12; :Trait13],  
    eltypes = [UTF8String; UTF8String; Float64; Float64; Float64; Float64; Float64; 
               Float64; Float64; Float64; Float64; Float64; Float64; Float64; Float64])
```




<table class="data-frame"><tr><th></th><th>FID</th><th>IID</th><th>Trait1</th><th>Trait2</th><th>Trait3</th><th>Trait4</th><th>Trait5</th><th>Trait6</th><th>Trait7</th><th>Trait8</th><th>Trait9</th><th>Trait10</th><th>Trait11</th><th>Trait12</th><th>Trait13</th></tr><tr><th>1</th><td>10002K</td><td>10002K</td><td>-1.81573145026234</td><td>-0.94615046147283</td><td>1.11363077580442</td><td>-2.09867121119159</td><td>0.744416614111748</td><td>0.00139171884080131</td><td>0.934732480409667</td><td>-1.22677315418103</td><td>1.1160784277875</td><td>-0.4436280335029</td><td>0.824465656443384</td><td>-1.02852542216546</td><td>-0.394049201727681</td></tr><tr><th>2</th><td>10004O</td><td>10004O</td><td>-1.24440094378729</td><td>0.109659992547179</td><td>0.467119394241789</td><td>-1.62131304097589</td><td>1.0566758355683</td><td>0.978946979419181</td><td>1.00014633946047</td><td>0.32487427140228</td><td>1.16232175219696</td><td>2.6922706948705</td><td>3.08263672461047</td><td>1.09064954786013</td><td>0.0256616415357438</td></tr><tr><th>3</th><td>10005Q</td><td>10005Q</td><td>1.45566914502305</td><td>1.53866932923243</td><td>1.09402959376555</td><td>0.586655272226893</td><td>-0.32796454430367</td><td>-0.30337709778827</td><td>-0.0334354881314741</td><td>-0.464463064285437</td><td>-0.3319396273436</td><td>-0.486839089635991</td><td>-1.10648681564373</td><td>-1.42015780427231</td><td>-0.687463456644413</td></tr><tr><th>4</th><td>10006S</td><td>10006S</td><td>-0.768809276698548</td><td>0.513490885514249</td><td>0.244263028382142</td><td>-1.31740254475691</td><td>1.19393774326845</td><td>1.17344127734288</td><td>1.08737426675232</td><td>0.536022583732261</td><td>0.802759240762068</td><td>0.234159411749815</td><td>0.394174866891074</td><td>-0.767365892476029</td><td>0.0635385761884935</td></tr><tr><th>5</th><td>10009Y</td><td>10009Y</td><td>-0.264415132547719</td><td>-0.348240421825694</td><td>-0.0239065083413606</td><td>0.00473915802244948</td><td>1.25619191712193</td><td>1.2038883667631</td><td>1.29800739042627</td><td>0.310113660247311</td><td>0.626159861059352</td><td>0.899289129831224</td><td>0.54996783350812</td><td>0.540687809542048</td><td>0.179675416046033</td></tr><tr><th>6</th><td>10010J</td><td>10010J</td><td>-1.37617270917293</td><td>-1.47191967744564</td><td>0.291179894254146</td><td>-0.803110740704731</td><td>-0.264239977442213</td><td>-0.260573027836772</td><td>-0.165372266287781</td><td>-0.219257294118362</td><td>1.04702422290318</td><td>-0.0985815534616482</td><td>0.947393438068448</td><td>0.594014812031438</td><td>0.245407436348479</td></tr><tr><th>7</th><td>10011L</td><td>10011L</td><td>0.1009416296374</td><td>-0.191615722103455</td><td>-0.567421321596677</td><td>0.378571487240382</td><td>-0.246656179817904</td><td>-0.608810750053858</td><td>0.189081058215596</td><td>-1.27077787326519</td><td>-0.452476199143965</td><td>0.702562877297724</td><td>0.332636218957179</td><td>0.0026916503626181</td><td>0.317117176705358</td></tr><tr><th>8</th><td>10013P</td><td>10013P</td><td>-0.319818276367464</td><td>1.35774480657283</td><td>0.818689545938528</td><td>-1.15565531644352</td><td>0.63448368102259</td><td>0.291461908634679</td><td>0.933323714954726</td><td>-0.741083289682492</td><td>0.647477683507572</td><td>-0.970877627077966</td><td>0.220861165411304</td><td>0.852512250237764</td><td>-0.225904624283945</td></tr><tr><th>9</th><td>10014R</td><td>10014R</td><td>-0.288334173342032</td><td>0.566082538090752</td><td>0.254958336116175</td><td>-0.652578302869714</td><td>0.668921559277347</td><td>0.978309199170558</td><td>0.122862966041938</td><td>1.4790926378214</td><td>0.0672132424173449</td><td>0.0795903917527827</td><td>0.167532455243232</td><td>0.246915579442139</td><td>0.539932616458363</td></tr><tr><th>10</th><td>10015T</td><td>10015T</td><td>-1.15759732583991</td><td>-0.781198583545165</td><td>-0.595807759833517</td><td>-1.00554980260402</td><td>0.789828885933321</td><td>0.571058413379044</td><td>0.951304176233755</td><td>-0.295962982984816</td><td>0.99042002479707</td><td>0.561309366988983</td><td>0.733100030623233</td><td>-1.73467772245684</td><td>-1.35278484330654</td></tr><tr><th>11</th><td>10017X</td><td>10017X</td><td>0.740569150459031</td><td>1.40873846755415</td><td>0.734689999440088</td><td>0.0208322841295094</td><td>-0.337440968561619</td><td>-0.458304040611395</td><td>-0.142582512772326</td><td>-0.580392297464107</td><td>-0.684684998101516</td><td>-0.00785381461893456</td><td>-0.712244337518008</td><td>-0.313345561230878</td><td>-0.345419463162219</td></tr><tr><th>12</th><td>10020M</td><td>10020M</td><td>-0.675892486454995</td><td>0.279892613829682</td><td>0.267915996308248</td><td>-1.04103665392985</td><td>0.910741715645888</td><td>0.866027618513171</td><td>1.07414431702005</td><td>0.0381751003538302</td><td>0.766355377018601</td><td>-0.340118016143495</td><td>-0.809013958505059</td><td>0.548521663785885</td><td>-0.0201828675962336</td></tr><tr><th>13</th><td>10022Q</td><td>10022Q</td><td>-0.795410435603455</td><td>-0.699989939762738</td><td>0.3991295030063</td><td>-0.510476261900736</td><td>1.51552245416844</td><td>1.28743032939467</td><td>1.53772393250903</td><td>0.133989160117702</td><td>1.02025736886037</td><td>0.499018733899186</td><td>-0.36948273277931</td><td>-1.10153460436318</td><td>-0.598132438886619</td></tr><tr><th>14</th><td>10023S</td><td>10023S</td><td>-0.193483122930324</td><td>-0.286021160323518</td><td>-0.691494225262995</td><td>0.0131581678700699</td><td>1.52337470686782</td><td>1.4010638072262</td><td>1.53114620451896</td><td>0.333066483478075</td><td>1.04372480381099</td><td>0.163206783570466</td><td>-0.422883765001728</td><td>-0.383527976713573</td><td>-0.489221907788158</td></tr><tr><th>15</th><td>10028C</td><td>10028C</td><td>0.151246203379718</td><td>2.09185108993614</td><td>2.03800472474384</td><td>-1.12474717143531</td><td>1.66557024390713</td><td>1.62535675109576</td><td>1.58751070483655</td><td>0.635852186043776</td><td>0.842577784605979</td><td>0.450761870778952</td><td>-1.39479033623028</td><td>-0.560984107567768</td><td>0.289349776549287</td></tr><tr><th>16</th><td>10031R</td><td>10031R</td><td>-0.464608740812712</td><td>0.36127694772303</td><td>1.2327673928287</td><td>-0.826033731086383</td><td>1.43475224709983</td><td>1.74451823818846</td><td>0.211096887484638</td><td>2.64816425140548</td><td>1.02511433146096</td><td>0.11975731603184</td><td>0.0596832073448267</td><td>-0.631231612661616</td><td>-0.207878671782927</td></tr><tr><th>17</th><td>10032T</td><td>10032T</td><td>-0.732977488012215</td><td>-0.526223425889779</td><td>0.61657871336593</td><td>-0.55447974332593</td><td>0.947484859025104</td><td>0.936833214138173</td><td>0.972516806335524</td><td>0.290251013865227</td><td>1.01285359725723</td><td>0.516207422283291</td><td>-0.0300689171988194</td><td>0.8787322524583</td><td>0.450254629309513</td></tr><tr><th>18</th><td>10034X</td><td>10034X</td><td>-0.167326459622119</td><td>0.175327165487237</td><td>0.287467725892572</td><td>-0.402652532084246</td><td>0.551181509418056</td><td>0.522204743290975</td><td>0.436837660094653</td><td>0.299564933845579</td><td>0.583109520896067</td><td>-0.704415820005353</td><td>-0.730810367994577</td><td>-1.95140580379896</td><td>-0.933504665700164</td></tr><tr><th>19</th><td>10035Z</td><td>10035Z</td><td>1.41159485787418</td><td>1.78722407901017</td><td>0.84397639585364</td><td>0.481278083772991</td><td>-0.0887673728508268</td><td>-0.49957757426858</td><td>0.304195897924847</td><td>-1.23884208383369</td><td>-0.153475724036624</td><td>-0.870486102788329</td><td>0.0955473331150403</td><td>-0.983708050882817</td><td>-0.3563445644514</td></tr><tr><th>20</th><td>10041U</td><td>10041U</td><td>-1.42997091652825</td><td>-0.490147045034213</td><td>0.272730237607695</td><td>-1.61029992954153</td><td>0.990787817197748</td><td>0.711687532608184</td><td>1.1885836012715</td><td>-0.371229188075638</td><td>1.24703459239952</td><td>-0.0389162332271516</td><td>0.883495749072872</td><td>2.58988026321017</td><td>3.33539552370368</td></tr><tr><th>21</th><td>10047G</td><td>10047G</td><td>-0.147247288176765</td><td>0.12328430415652</td><td>0.617549051912237</td><td>-0.18713077178262</td><td>0.256438107586694</td><td>0.17794983735083</td><td>0.412611806463263</td><td>-0.244809124559737</td><td>0.0947624806136492</td><td>0.723017223849532</td><td>-0.683948354633436</td><td>0.0873751276309269</td><td>-0.262209652750371</td></tr><tr><th>22</th><td>10051X</td><td>10051X</td><td>-0.187112676773894</td><td>-0.270777264595619</td><td>-1.01556818551606</td><td>0.0602850568600233</td><td>0.272419757757978</td><td>0.869133161879197</td><td>-0.657519461414234</td><td>2.32388522018189</td><td>-0.999936011525034</td><td>1.44671844178306</td><td>0.971157886040772</td><td>-0.358747904241515</td><td>-0.439657942096136</td></tr><tr><th>23</th><td>10052Z</td><td>10052Z</td><td>-1.82434047163768</td><td>-0.933480446068067</td><td>1.29474003766977</td><td>-1.94545221151036</td><td>0.33584651189654</td><td>0.359201654302844</td><td>0.513652924365886</td><td>-0.073197696696958</td><td>1.57139042812005</td><td>1.53329371326728</td><td>1.82076821859528</td><td>2.22740301867829</td><td>1.50063347195857</td></tr><tr><th>24</th><td>10056H</td><td>10056H</td><td>-2.29344084351335</td><td>-2.49161842344418</td><td>0.40383988742336</td><td>-2.36488074752948</td><td>1.4105254831956</td><td>1.42244117147792</td><td>1.17024166272172</td><td>0.84476650176855</td><td>1.79026875432495</td><td>0.648181858970515</td><td>-0.0857231057403538</td><td>-1.02789535292617</td><td>0.491288088952859</td></tr><tr><th>25</th><td>10057J</td><td>10057J</td><td>-0.434135932888305</td><td>0.740881989034652</td><td>0.699576357578518</td><td>-1.02405543187775</td><td>0.759529223983713</td><td>0.956656110895288</td><td>0.633299568656589</td><td>0.770733932268516</td><td>0.824988511714526</td><td>1.84287437634769</td><td>1.91045942063443</td><td>-0.502317207869366</td><td>0.132670133448219</td></tr><tr><th>26</th><td>10058L</td><td>10058L</td><td>-2.1920969546557</td><td>-2.49465664272271</td><td>0.354854763893431</td><td>-1.93155848635714</td><td>0.941979400289938</td><td>0.978917101414106</td><td>0.894860097289736</td><td>0.463239402831873</td><td>1.12537133317163</td><td>1.70528446191955</td><td>0.717792714479123</td><td>0.645888049108261</td><td>0.783968250169388</td></tr><tr><th>27</th><td>10060Y</td><td>10060Y</td><td>-1.46602269088422</td><td>-1.24921677101897</td><td>0.307977693653039</td><td>-1.55097364660989</td><td>0.618908494474798</td><td>0.662508171662042</td><td>0.475957173906078</td><td>0.484718674597707</td><td>0.401564892028249</td><td>0.55987973254026</td><td>-0.376938143754217</td><td>-0.933982629228218</td><td>0.390013151672955</td></tr><tr><th>28</th><td>10062C</td><td>10062C</td><td>-1.83317744236881</td><td>-1.53268787828701</td><td>2.55674262685865</td><td>-1.51827745783835</td><td>0.789409601746455</td><td>0.908747799728588</td><td>0.649971922941479</td><td>0.668373649931667</td><td>1.20058303519903</td><td>0.277963256075637</td><td>1.2504953198275</td><td>3.31370445071638</td><td>2.22035828885342</td></tr><tr><th>29</th><td>10064G</td><td>10064G</td><td>-0.784546628243178</td><td>0.276582579543931</td><td>3.01104958800057</td><td>-1.11978843206758</td><td>0.920823858422707</td><td>0.750217689886151</td><td>1.26153730009639</td><td>-0.403363882922417</td><td>0.400667296857811</td><td>-0.217597941303479</td><td>-0.724669537565068</td><td>-0.391945338467193</td><td>-0.650023936358253</td></tr><tr><th>30</th><td>10065I</td><td>10065I</td><td>0.464455916345135</td><td>1.3326356122229</td><td>-1.23059563374303</td><td>-0.357975958937414</td><td>1.18249746977104</td><td>1.54315938069757</td><td>-0.60339041154062</td><td>3.38308845958422</td><td>0.823740765148641</td><td>-0.129951318508883</td><td>-0.657979878422938</td><td>-0.499534924074273</td><td>-0.414476569095651</td></tr><tr><th>&vellip;</th><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td></tr></table>




```julia
#describe(cg10k_trait)
```


```julia
Y = convert(Matrix{Float64}, cg10k_trait[:, 3:15])
histogram(Y, layout = 13)
```




<img src="data:image/png;base64,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" />



## Pre-processing data for heritability analysis

To prepare variance component model fitting, we form an instance of `VarianceComponentVariate`. The two variance components are $(2\Phi, I)$.


```julia
using VarianceComponentModels

# form data as VarianceComponentVariate
cg10kdata = VarianceComponentVariate(Y, (2Φgrm, eye(size(Y, 1))))
fieldnames(cg10kdata)
```




    3-element Array{Symbol,1}:
     :Y
     :X
     :V




```julia
cg10kdata
```




    VarianceComponentModels.VarianceComponentVariate{Float64,2,Array{Float64,2},Array{Float64,2},Array{Float64,2}}(6670x13 Array{Float64,2}:
     -1.81573   -0.94615     1.11363    …   0.824466  -1.02853     -0.394049 
     -1.2444     0.10966     0.467119       3.08264    1.09065      0.0256616
      1.45567    1.53867     1.09403       -1.10649   -1.42016     -0.687463 
     -0.768809   0.513491    0.244263       0.394175  -0.767366     0.0635386
     -0.264415  -0.34824    -0.0239065      0.549968   0.540688     0.179675 
     -1.37617   -1.47192     0.29118    …   0.947393   0.594015     0.245407 
      0.100942  -0.191616   -0.567421       0.332636   0.00269165   0.317117 
     -0.319818   1.35774     0.81869        0.220861   0.852512    -0.225905 
     -0.288334   0.566083    0.254958       0.167532   0.246916     0.539933 
     -1.1576    -0.781199   -0.595808       0.7331    -1.73468     -1.35278  
      0.740569   1.40874     0.73469    …  -0.712244  -0.313346    -0.345419 
     -0.675892   0.279893    0.267916      -0.809014   0.548522    -0.0201829
     -0.79541   -0.69999     0.39913       -0.369483  -1.10153     -0.598132 
      ⋮                                 ⋱   ⋮                                
     -0.131005   0.425378   -1.09015        0.35674    0.456428     0.882577 
     -0.52427    1.04173     1.13749        0.366737   1.78286      1.90764  
      1.32516    0.905899    0.84261    …  -0.418756  -0.275519    -0.912778 
     -1.44368   -2.55708    -0.868193       1.31914   -1.44981     -1.77373  
     -1.8518    -1.25726     1.81724        0.770329  -0.0470789    1.50496  
     -0.810034   0.0896703   0.530939       0.757479   1.10001      1.29115  
     -1.22395   -1.48953    -2.95847        1.29209    0.697478     0.228819 
     -0.282847  -1.54129    -1.38819    …   1.00973   -0.362158    -1.55022  
      0.475008   1.46697     0.497403       0.141684   0.183218     0.122664 
     -0.408154  -0.325323    0.0850869     -0.2214    -0.575183     0.399583 
      0.886626   0.487408   -0.0977307     -0.985545  -0.636874    -0.439825 
     -1.24394    0.213697    2.74965        1.39201    0.299931     0.392809 ,6670x0 Array{Float64,2},(
    6670x6670 Array{Float64,2}:
      1.00583       0.00659955   -0.000232427  …  -0.000129257  -0.00562459 
      0.00659955    0.99784      -0.00403985       0.00181974    0.00691145 
     -0.000232427  -0.00403985    0.987264         0.00058913   -0.000699707
      0.00186795   -0.00640781   -0.00372219      -0.00483365   -0.00254155 
     -0.000155086  -0.00721501    0.00362883       0.00427952   -0.00316764 
      0.00400741    0.00115477    0.005091     …   0.00188751   -3.65987e-6 
      0.00111701    0.00482842   -0.00375641       0.00243399   -0.00247849 
     -0.00131899    0.00639975   -0.00202991       0.00707293   -0.00048186 
     -0.00205238   -0.00240896   -0.00110924       0.00351173    0.00363799 
     -0.00273677    0.00423992    0.000238256     -0.0029461    -0.00210478 
     -0.00412287    0.000297635  -0.000950353  …  -0.000531045  -0.00212246 
      0.00190203    0.00334083    0.0036709       -0.00140732   -0.00626668 
      0.000660883  -0.00180829    0.00602955       0.00150954   -0.00254826 
      ⋮                                        ⋱                            
      0.00602273    0.00232083    0.00200852       1.33451e-5    0.00614137 
     -0.00428016    0.0054185    -0.00370108      -0.00219871    0.00733631 
      0.00109348   -0.00485292   -0.00610528   …  -0.00125803    0.00421559 
     -0.00845106   -0.00414261   -0.00218104      -0.00141161   -0.00101611 
     -0.00636811   -0.0015077     0.00624753       0.00105766   -7.21938e-5 
      0.000860393  -0.00394326    0.0053709       -0.0126635    -0.0104067  
      0.00442858    0.00169958   -0.00202223      -0.00188626   -0.00124884 
     -0.0045805    -0.000261196   0.000203706  …   0.00168027   -0.00460447 
     -0.00405834    0.00466013   -0.00262013       0.00395595   -0.00102754 
     -0.00192981   -0.00174465   -0.000141344      0.00249404   -0.00591688 
     -0.000129257   0.00181974    0.00058913       1.00197       0.00105123 
     -0.00562459    0.00691145   -0.000699707      0.00105123    1.00158    ,
    
    6670x6670 Array{Float64,2}:
     1.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  …  0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  1.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  1.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  1.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  1.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  1.0  0.0  0.0  …  0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  1.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  1.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  …  0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     ⋮                        ⋮              ⋱            ⋮                      
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  …  0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     1.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  1.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  …  0.0  0.0  1.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  1.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  1.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  1.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  1.0))



Before fitting the variance component model, we pre-compute the eigen-decomposition of $2\Phi_{\text{GRM}}$, the rotated responses, and the constant part in log-likelihood, and store them as a `TwoVarCompVariateRotate` instance, which is re-used in various variane component estimation procedures.


```julia
# pre-compute eigen-decomposition (~50 secs on my laptop)
@time cg10kdata_rotated = TwoVarCompVariateRotate(cg10kdata)
fieldnames(cg10kdata_rotated)
```

     51.297311 seconds (957.07 k allocations: 1.038 GB, 0.32% gc time)





    4-element Array{Symbol,1}:
     :Yrot    
     :Xrot    
     :eigval  
     :logdetV2



## Save intermediate results

We don't want to re-compute SnpArray and empirical kinship matrices again and again for heritibility analysis.


```julia
#using JLD
#@save "cg10k.jld"
#whos()
```

To load workspace


```julia
#using SnpArrays, JLD, DataFrames, VarianceComponentModels, Plots
#gr()
#@load "cg10k.jld"
#whos()
```

## Heritability of single traits

We use Fisher scoring algorithm to fit variance component model for each single trait.


```julia
# heritability from single trait analysis
hST = zeros(13)
# standard errors of estimated heritability
hST_se = zeros(13)
# additive genetic effects
σ2a = zeros(13)
# enviromental effects
σ2e = zeros(13)

@time for trait in 1:13
    println(names(cg10k_trait)[trait + 2])
    # form data set for trait j
    traitj_data = TwoVarCompVariateRotate(cg10kdata_rotated.Yrot[:, trait], cg10kdata_rotated.Xrot, 
        cg10kdata_rotated.eigval, cg10kdata_rotated.logdetV2)
    # initialize model parameters
    traitj_model = VarianceComponentModel(traitj_data)
    # estimate variance components
    _, _, _, Σcov, _, _ = mle_fs!(traitj_model, traitj_data; solver=:Ipopt, verbose=false)
    σ2a[trait] = traitj_model.Σ[1][1]
    σ2e[trait] = traitj_model.Σ[2][1]
    @show σ2a[trait], σ2e[trait]
    h, hse = heritability(traitj_model.Σ, Σcov)
    hST[trait] = h[1]
    hST_se[trait] = hse[1]
end
```

    Trait1
    
    ******************************************************************************
    This program contains Ipopt, a library for large-scale nonlinear optimization.
     Ipopt is released as open source code under the Eclipse Public License (EPL).
             For more information visit http://projects.coin-or.org/Ipopt
    ******************************************************************************
    
    (σ2a[trait],σ2e[trait]) = (0.26104123222538084,0.735688443211474)
    Trait2
    (σ2a[trait],σ2e[trait]) = (0.18874147373002487,0.8106899992330557)
    Trait3
    (σ2a[trait],σ2e[trait]) = (0.318571927651206,0.6801458862910434)
    Trait4
    (σ2a[trait],σ2e[trait]) = (0.26556901335366295,0.730358836480852)
    Trait5
    (σ2a[trait],σ2e[trait]) = (0.28123321199488277,0.7167989046615363)
    Trait6
    (σ2a[trait],σ2e[trait]) = (0.2829461159340443,0.7165629525047605)
    Trait7
    (σ2a[trait],σ2e[trait]) = (0.21543856399750066,0.7816211121996341)
    Trait8
    (σ2a[trait],σ2e[trait]) = (0.19412648726333176,0.805527765060679)
    Trait9
    (σ2a[trait],σ2e[trait]) = (0.24789561129713544,0.7504615853385027)
    Trait10
    (σ2a[trait],σ2e[trait]) = (0.100074557877669,0.8998152776356187)
    Trait11
    (σ2a[trait],σ2e[trait]) = (0.16486778143457112,0.8338002259854702)
    Trait12
    (σ2a[trait],σ2e[trait]) = (0.0829866038383105,0.9158035671624248)
    Trait13
    (σ2a[trait],σ2e[trait]) = (0.056842480167674306,0.9423653389084629)
      5.952183 seconds (49.17 M allocations: 1.102 GB, 5.89% gc time)



```julia
# heritability and standard errors
[hST'; hST_se']
```




    2x13 Array{Float64,2}:
     1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0
     1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0



## Pairwise traits

Following code snippet calculates joint heritability among all pairs of traits, a total of 78 bivariate variane component models.


```julia
# additive genetic effects (2x2 psd matrices) from bavariate trait analysis;
Σa = Array{Matrix{Float64}}(13, 13)
# environmental effects (2x2 psd matrices) from bavariate trait analysis;
Σe = Array{Matrix{Float64}}(13, 13)

@time for i in 1:13
    for j in (i+1):13
        println(names(cg10k_trait)[i + 2], names(cg10k_trait)[j + 2])
        # form data set for (trait1, trait2)
        traitij_data = TwoVarCompVariateRotate(cg10kdata_rotated.Yrot[:, [i;j]], cg10kdata_rotated.Xrot, 
            cg10kdata_rotated.eigval, cg10kdata_rotated.logdetV2)
        # initialize model parameters
        traitij_model = VarianceComponentModel(traitij_data)
        # estimate variance components
        mle_fs!(traitij_model, traitij_data; solver=:Ipopt, verbose=false)
        Σa[i, j] = traitij_model.Σ[1]
        Σe[i, j] = traitij_model.Σ[2]
        @show Σa[i, j], Σe[i, j]
    end
end
```

    Trait1Trait2
    (Σa[i,j],Σe[i,j]) = (
    [0.26011943486815303 0.1762158250641666
     0.1762158250641666 0.18737615484318682],
    
    [0.7365894055240527 0.5838920954583557
     0.5838920954583557 0.8120331390255984])
    Trait1Trait3
    (Σa[i,j],Σe[i,j]) = (
    [0.2615639935568583 -0.013126818536804538
     -0.013126818536804538 0.3190566225477499],
    
    [0.7351802111593627 -0.12112674834322068
     -0.12112674834322068 0.6796789899239241])
    Trait1Trait4
    (Σa[i,j],Σe[i,j]) = (
    [0.26087960315961645 0.2226144055987542
     0.2226144055987542 0.26558083276300853],
    
    [0.735845998164204 0.5994353345857374
     0.5994353345857374 0.7303474474479689])
    Trait1Trait5
    (Σa[i,j],Σe[i,j]) = (
    [0.26078303770342093 -0.14701178378027208
     -0.14701178378027208 0.2818772454637467],
    
    [0.7359373011280201 -0.2545838905575993
     -0.2545838905575993 0.7161761242026842])
    Trait1Trait6
    (Σa[i,j],Σe[i,j]) = (
    [0.26070703552005153 -0.1293564259272152
     -0.1293564259272152 0.28318838486326464],
    
    [0.7360128807191256 -0.23136128330673647
     -0.23136128330673647 0.7163294323420235])
    Trait1Trait7
    (Σa[i,j],Σe[i,j]) = (
    [0.26030750074865355 -0.1402575370658552
     -0.1402575370658552 0.2150805562437876],
    
    [0.736405599875614 -0.19780547644886934
     -0.19780547644886934 0.7819851899273343])
    Trait1Trait8
    (Σa[i,j],Σe[i,j]) = (
    [0.2610345999109415 -0.033529628072184174
     -0.033529628072184174 0.1941430731435061],
    
    [0.7356949687052194 -0.12627246367045894
     -0.12627246367045894 0.8055115370752971])
    Trait1Trait9
    (Σa[i,j],Σe[i,j]) = (
    [0.26301631599826725 -0.20486492716403212
     -0.20486492716403212 0.24679565235839057],
    
    [0.733794462308188 -0.30745013667457605
     -0.30745013667457605 0.7515442213436087])
    Trait1Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.26089807908154783 -0.09981756181315272
     -0.09981756181315272 0.0970232854366607],
    
    [0.7358279769596973 -0.3036087596209461
     -0.3036087596209461 0.902853465186668])
    Trait1Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.2607397076720635 -0.1389834153966234
     -0.1389834153966234 0.16306263185657766],
    
    [0.7359820027777678 -0.3591745321509024
     -0.3591745321509024 0.8355950431882248])
    Trait1Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.26306860004688026 -0.14553554987243783
     -0.14553554987243783 0.08051358652010815],
    
    [0.7337809604103112 -0.04169750224640395
     -0.04169750224640395 0.9183594400218908])
    Trait1Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.26234367608192977 -0.10889551713434968
     -0.10889551713434968 0.0512940383653612],
    
    [0.7344496461752492 -0.11399558207532524
     -0.11399558207532524 0.9479424007586392])
    Trait2Trait3
    (Σa[i,j],Σe[i,j]) = (
    [0.18901532602954713 0.14615743011868396
     0.14615743011868396 0.32052865893932325],
    
    [0.8104184413491479 0.09749923852720849
     0.09749923852720849 0.6782713240476694])
    Trait2Trait4
    (Σa[i,j],Σe[i,j]) = (
    [0.1883951499036305 0.07521464811301082
     0.07521464811301082 0.26555848041382424],
    
    [0.8110301028366897 0.22049483157599306
     0.22049483157599306 0.7303691342557705])
    Trait2Trait5
    (Σa[i,j],Σe[i,j]) = (
    [0.18871644002070298 -0.011314018224773681
     -0.011314018224773681 0.2812465335272833],
    
    [0.8107145247899494 -0.03701047017353238
     -0.03701047017353238 0.7167859986683998])
    Trait2Trait6
    (Σa[i,j],Σe[i,j]) = (
    [0.1887737598357356 -0.003106603698693131
     -0.003106603698693131 0.28301251325921034],
    
    [0.8106583657770059 -0.021182656859616053
     -0.021182656859616053 0.7164985874165493])
    Trait2Trait7
    (Σa[i,j],Σe[i,j]) = (
    [0.1883522257152029 -0.02995792853329811
     -0.02995792853329811 0.21518854249006697],
    
    [0.8110719442645782 -0.00136938653417689
     -0.00136938653417689 0.7818678818050632])
    Trait2Trait8
    (Σa[i,j],Σe[i,j]) = (
    [0.18926168900224644 0.033142298438513934
     0.033142298438513934 0.19466629556574452],
    
    [0.8101822287413256 -0.03260027070436449
     -0.03260027070436449 0.8050045055360936])
    Trait2Trait9
    (Σa[i,j],Σe[i,j]) = (
    [0.1872848956280239 -0.08541458777076169
     -0.08541458777076169 0.24671880340715757],
    
    [0.8121330455990424 -0.08087908481224644
     -0.08087908481224644 0.7516171286963302])
    Trait2Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.1889645629678981 -0.12531880400449946
     -0.12531880400449946 0.10012137188898965],
    
    [0.8104983819152255 -0.2710710218717953
     -0.2710710218717953 0.8998490679544218])
    Trait2Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.18776200371995733 -0.11847920330746468
     -0.11847920330746468 0.16627341912957957],
    
    [0.8116528153514335 -0.29554899494751774
     -0.29554899494751774 0.8324372717265173])
    Trait2Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.18819063520893153 -0.09053833116231333
     -0.09053833116231333 0.08226341390714831],
    
    [0.8112716597628467 0.04542203421221531
     0.04542203421221531 0.9165863321400829])
    Trait2Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.18826030571365 -0.07070412373503615
     -0.07070412373503615 0.05472389418108183],
    
    [0.8112166105461245 0.07379770159309924
     0.07379770159309924 0.9445208397201135])
    Trait3Trait4
    (Σa[i,j],Σe[i,j]) = (
    [0.31852039540025556 -0.15433893723722844
     -0.15433893723722844 0.26475409905630554],
    
    [0.6801958865832615 -0.3034399519741299
     -0.3034399519741299 0.731151851539042])
    Trait3Trait5
    (Σa[i,j],Σe[i,j]) = (
    [0.31896997787115844 0.1843544667615588
     0.1843544667615588 0.2825001772391643],
    
    [0.6797599597904634 0.3364105248325953
     0.3364105248325953 0.7155665412290728])
    Trait3Trait6
    (Σa[i,j],Σe[i,j]) = (
    [0.3195663644223704 0.16663988772541583
     0.16663988772541583 0.2850313182313418],
    
    [0.6791832508340369 0.2976976595424014
     0.2976976595424014 0.7145358499994827])
    Trait3Trait7
    (Σa[i,j],Σe[i,j]) = (
    [0.3185755051463484 0.16685216000557962
     0.16685216000557962 0.21523224226058452],
    
    [0.6801424314852816 0.3471388423837074
     0.3471388423837074 0.781823130994699])
    Trait3Trait8
    (Σa[i,j],Σe[i,j]) = (
    [0.32049923480743736 0.05753194037721402
     0.05753194037721402 0.19724489854138733],
    
    [0.6782830498088369 0.04425974188401513
     0.04425974188401513 0.802473783266528])
    Trait3Trait9
    (Σa[i,j],Σe[i,j]) = (
    [0.31871912001262215 0.13729240537832613
     0.13729240537832613 0.24697586633940616],
    
    [0.6800039145595509 0.2671054378272429
     0.2671054378272429 0.7513573840671314])
    Trait3Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.3189152132179772 -0.0786338234436915
     -0.0786338234436915 0.10110317193796312],
    
    [0.679814558884657 -0.14078871656902353
     -0.14078871656902353 0.8987982713072635])
    Trait3Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.3178223304600456 -0.01798395917145184
     -0.01798395917145184 0.16474292116632128],
    
    [0.6808712744789884 -0.11416573111718417
     -0.11416573111718417 0.8339228729000641])
    Trait3Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.3208883401716964 0.08452483760009306
     0.08452483760009306 0.0869867503171365],
    
    [0.6779139477279349 0.0340132717811355
     0.0340132717811355 0.9118411342532936])
    Trait3Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.32300879330585164 0.11068106826965814
     0.11068106826965814 0.06117389943402113],
    
    [0.6759011385390731 -0.007296623864558913
     -0.007296623864558913 0.9380722549857157])
    Trait4Trait5
    (Σa[i,j],Σe[i,j]) = (
    [0.26566699036169605 -0.2158475608286269
     -0.2158475608286269 0.28291866201101923],
    
    [0.7302544030223196 -0.376675282891704
     -0.376675282891704 0.7151643527045644])
    Trait4Trait6
    (Σa[i,j],Σe[i,j]) = (
    [0.26614305300873853 -0.20063378204267554
     -0.20063378204267554 0.2844419499822279],
    
    [0.7297943215353132 -0.3468040727719556
     -0.3468040727719556 0.7151119490174423])
    Trait4Trait7
    (Σa[i,j],Σe[i,j]) = (
    [0.2644898029398779 -0.18275157344131776
     -0.18275157344131776 0.2141168002388134],
    
    [0.7314145610167773 -0.3261719955266139
     -0.3261719955266139 0.7829351279927492])
    Trait4Trait8
    (Σa[i,j],Σe[i,j]) = (
    [0.26669395421675546 -0.09763540115429367
     -0.09763540115429367 0.1961260872033616],
    
    [0.7292655955847159 -0.15036048225408044
     -0.15036048225408044 0.8035709853965969])
    Trait4Trait9
    (Σa[i,j],Σe[i,j]) = (
    [0.27003652025292474 -0.22740698731918993
     -0.22740698731918993 0.2480458221785815],
    
    [0.7260245141067312 -0.4156014356566848
     -0.4156014356566848 0.7502976667276586])
    Trait4Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.2655429418281044 -0.03381072154126782
     -0.03381072154126782 0.0996098263599129],
    
    [0.7303952879665676 -0.22772490049078267
     -0.22772490049078267 0.9002752447772486])
    Trait4Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.2656276022067445 -0.0967400332453423
     -0.0967400332453423 0.16327409086062875],
    
    [0.7303019079378218 -0.2726111957424877
     -0.2726111957424877 0.8353713025165547])
    Trait4Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.26816369655143846 -0.14161261171741682
     -0.14161261171741682 0.08039677346947334],
    
    [0.7278825933041443 -0.08284654589719648
     -0.08284654589719648 0.9184455609265872])
    Trait4Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.26617123205334886 -0.09807308729740324
     -0.09807308729740324 0.05401019174697443],
    
    [0.7297749332697795 -0.22505950767079727
     -0.22505950767079727 0.9452044744732253])
    Trait5Trait6
    (Σa[i,j],Σe[i,j]) = (
    [0.2815916529117581 0.28089837189876277
     0.28089837189876277 0.2822980504457378],
    
    [0.7164553609388885 0.6603676496624155
     0.6603676496624155 0.7171950644786848])
    Trait5Trait7
    (Σa[i,j],Σe[i,j]) = (
    [0.28081437389685465 0.2320998689028785
     0.2320998689028785 0.21166204276578496],
    
    [0.7172180317128432 0.6743038172539134
     0.6743038172539134 0.7853426270933965])
    Trait5Trait8
    (Σa[i,j],Σe[i,j]) = (
    [0.28134012176547224 0.16394787427644833
     0.16394787427644833 0.19270331039789454],
    
    [0.716700910916717 0.22103210912788746
     0.22103210912788746 0.8069220956459742])
    Trait5Trait9
    (Σa[i,j],Σe[i,j]) = (
    [0.2838778024864278 0.24453172568336923
     0.24453172568336923 0.24129303734496713],
    
    [0.7142441395374184 0.5084169870279653
     0.5084169870279653 0.75689423383449])
    Trait5Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.2818130149579839 -0.04621141466318818
     -0.04621141466318818 0.10148069053702076],
    
    [0.7162383352810132 -0.05721113478873817
     -0.05721113478873817 0.8984242888549007])
    Trait5Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.28042354198276 0.020249545578616107
     0.020249545578616107 0.16400332024863942],
    
    [0.7175856258522051 -0.03524364742481622
     -0.03524364742481622 0.8346493308274309])
    Trait5Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.28142706977067977 0.0616130662199462
     0.0616130662199462 0.08271662802494333],
    
    [0.7166145561763498 0.05292864493625416
     0.05292864493625416 0.9160739286144334])
    Trait5Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.2822915336770548 0.07042205168959345
     0.07042205168959345 0.05694630997371182],
    
    [0.7157823114671132 0.05283744551208099
     0.05283744551208099 0.9422684292501169])
    Trait6Trait7
    (Σa[i,j],Σe[i,j]) = (
    [0.28296107605405046 0.22065632106005845
     0.22065632106005845 0.2138560977045212],
    
    [0.7165486719516274 0.5810829183804748
     0.5810829183804748 0.7831785715682008])
    Trait6Trait8
    (Σa[i,j],Σe[i,j]) = (
    [0.2829606227323868 0.18407962647041584
     0.18407962647041584 0.19237902250043631],
    
    [0.7165491133456647 0.4365973893626811
     0.4365973893626811 0.8072460050916037])
    Trait6Trait9
    (Σa[i,j],Σe[i,j]) = (
    [0.2849784406224258 0.23443573773563228
     0.23443573773563228 0.24320710809890078],
    
    [0.7146005305248684 0.4768263391920377
     0.4768263391920377 0.7550279957443258])
    Trait6Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.28365475689742714 -0.043548481043600644
     -0.043548481043600644 0.10202532158188515],
    
    [0.7158768651026055 -0.05916812562687212
     -0.05916812562687212 0.8978859540367762])
    Trait6Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.2815224136591653 0.027999198497497716
     0.027999198497497716 0.16342991310189497],
    
    [0.7179464769862334 -0.05241060710393953
     -0.05241060710393953 0.8352130875902175])
    Trait6Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.28311434441170164 0.05713989944414399
     0.05713989944414399 0.0826789313035068],
    
    [0.7164030333437527 0.04791987131194029
     0.04791987131194029 0.9161120298125991])
    Trait6Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.2838199671572152 0.06112120329707695
     0.06112120329707695 0.057081689708499625],
    
    [0.7157217792743793 0.0532698347458057
     0.0532698347458057 0.9421333545367362])
    Trait7Trait8
    (Σa[i,j],Σe[i,j]) = (
    [0.2138568636062993 0.08845552117563456
     0.08845552117563456 0.19237305507185362],
    
    [0.7831777634587772 -0.056833059546840724
     -0.056833059546840724 0.8072518674868492])
    Trait7Trait9
    (Σa[i,j],Σe[i,j]) = (
    [0.2187556960318311 0.21704186383319593
     0.21704186383319593 0.24414386504096938],
    
    [0.7784327750408729 0.4629009682465131
     0.4629009682465131 0.7541229287887962])
    Trait7Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.21627296245459482 -0.042114572555895034
     -0.042114572555895034 0.10209048393905551],
    
    [0.7808073891153632 -0.0859074527248701
     -0.0859074527248701 0.8978220136433211])
    Trait7Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.214068776570261 0.020696687537510253
     0.020696687537510253 0.16347380945463358],
    
    [0.7829608191256512 -0.04814801124193434
     -0.04814801124193434 0.8351704867201007])
    Trait7Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.21493232507095117 0.07578743223256132
     0.07578743223256132 0.08087309077615555],
    
    [0.7821316768241421 0.03469555448086824
     0.03469555448086824 0.9179149818382321])
    Trait7Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.21595864338688792 0.07493726652193226
     0.07493726652193226 0.054593411293268314],
    
    [0.7811387334966593 0.038905793654869604
     0.038905793654869604 0.9446215737969718])
    Trait8Trait9
    (Σa[i,j],Σe[i,j]) = (
    [0.1945549993508713 0.11281615771703897
     0.11281615771703897 0.247244151913094],
    
    [0.8051240570328738 0.18477843242872458
     0.18477843242872458 0.7510982278178284])
    Trait8Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.1944410026133888 -0.015634153645910635
     -0.015634153645910635 0.10042647889553495],
    
    [0.8052215712374999 0.011982589675582881
     0.011982589675582881 0.8994678453427947])
    Trait8Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.19385316433932662 0.02253246401737779
     0.02253246401737779 0.16466854383170343],
    
    [0.8057962945707323 -0.027274736865623306
     -0.027274736865623306 0.833996968298484])
    Trait8Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.19395121849388605 -0.00287601218605317
     -0.00287601218605317 0.08285727476214352],
    
    [0.805699796364685 0.03361278792998698
     0.03361278792998698 0.9159318672850477])
    Trait8Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.19397976155591182 0.004078666572513058
     0.004078666572513058 0.056907123220398724],
    
    [0.8056716012385241 0.037881707520808355
     0.037881707520808355 0.9423010776521313])
    Trait9Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.24729383232846244 -0.0023083632339188282
     -0.0023083632339188282 0.09982638783604271],
    
    [0.7510505981827851 0.07407294366362593
     0.07407294366362593 0.9000613327580784])
    Trait9Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.24782344372399562 0.03182350298547301
     0.03182350298547301 0.16489046411211802],
    
    [0.7505321390729256 0.15228537870520292
     0.15228537870520292 0.8337779553469964])
    Trait9Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.2503346951480958 0.08457136153453912
     0.08457136153453912 0.08875872341182169],
    
    [0.7480909649325541 0.1077563215164905
     0.1077563215164905 0.910108091340272])
    Trait9Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.24944189418695698 0.09348451303646096
     0.09348451303646096 0.05793201495098822],
    
    [0.7489745640198023 0.09821909823020973
     0.09821909823020973 0.9413348855377766])
    Trait10Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.09313966828035954 0.10003371877497542
     0.10003371877497542 0.16495492788508725],
    
    [0.9067034447680414 0.47442665936453843
     0.47442665936453843 0.8337151519497663])
    Trait10Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.09672471644137419 0.05640434659676373
     0.05640434659676373 0.07945282707687533],
    
    [0.9031497841187028 0.08532319187509019
     0.08532319187509019 0.91933434342665])
    Trait10Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.10098492172122948 -0.027991587710857326
     -0.027991587710857326 0.05783694113832583],
    
    [0.8989368215833939 0.16605077488640077
     0.16605077488640077 0.9413901799217301])
    Trait11Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.16384057742661823 0.05703178017251179
     0.05703178017251179 0.07921436807844635],
    
    [0.8348140972530623 0.14559650973720015
     0.14559650973720015 0.9195521322043868])
    Trait11Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.16488293337335094 -0.0015841372093299245
     -0.0015841372093299245 0.05749684375147145],
    
    [0.8337979488908841 0.2006122283482765
     0.2006122283482765 0.9417152320985648])
    Trait12Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.08314058460338657 -0.30414162669938277
     -0.30414162669938277 6.1516805202088326e7],
    
    [0.9156535422859348 0.20300255194828656
     0.20300255194828656 7.289623734162273e9])
     73.287758 seconds (3.33 G allocations: 50.885 GB, 10.69% gc time)


## 5-trait analysis

Researchers want to jointly analyze traits 5-9. Our strategy is to try both Fisher scoring and MM algorithm with different starting point, and choose the best local optimum. We first form the data set and run Fisher scoring, which yields a final objective value 8.6836898e+03.


```julia
traitidx = 5:9
# form data set
trait59_data = TwoVarCompVariateRotate(cg10kdata_rotated.Yrot[:, traitidx], cg10kdata_rotated.Xrot, 
    cg10kdata_rotated.eigval, cg10kdata_rotated.logdetV2)
# initialize model parameters
trait59_model = VarianceComponentModel(trait59_data)
# estimate variance components
@time mle_fs!(trait59_model, trait59_data; solver=:Knitro, verbose=true)
trait59_model
```

    
    Knitro 10.1.0 STUDENT LICENSE (problem size limit = 300)
    
    =======================================
                Student License
           (NOT FOR COMMERCIAL USE)
             Artelys Knitro 10.1.0
    =======================================
    
    Knitro presolve eliminated 0 variables and 0 constraints.
    
    algorithm:            1
    The problem is identified as bound constrained only.
    Knitro changing bar_initpt from AUTO to 3.
    Knitro changing bar_murule from AUTO to 4.
    Knitro changing bar_penaltycons from AUTO to 1.
    Knitro changing bar_penaltyrule from AUTO to 2.
    Knitro changing bar_switchrule from AUTO to 1.
    Knitro changing linsolver from AUTO to 2.
    
    Problem Characteristics                    ( Presolved)
    -----------------------
    Objective goal:  Maximize
    Number of variables:                    30 (        30)
        bounded below:                      10 (        10)
        bounded above:                       0 (         0)
        bounded below and above:             0 (         0)
        fixed:                               0 (         0)
        free:                               20 (        20)
    Number of constraints:                   0 (         0)
        linear equalities:                   0 (         0)
        nonlinear equalities:                0 (         0)
        linear inequalities:                 0 (         0)
        nonlinear inequalities:              0 (         0)
        range:                               0 (         0)
    Number of nonzeros in Jacobian:          0 (         0)
    Number of nonzeros in Hessian:         465 (       465)
    
      Iter      Objective      FeasError   OptError    ||Step||    CGits 
    --------  --------------  ----------  ----------  ----------  -------
           0   -5.042149e+04   0.000e+00
          10   -1.677917e+04   0.000e+00   1.102e+03   8.413e-02        0
          20   -1.531001e+03   0.000e+00   1.489e+04   2.670e-03        0
          30    8.342379e+03   0.000e+00   8.574e+03   4.155e-04        7
          40    8.638819e+03   0.000e+00   3.597e+03   7.576e-05       63
          50    8.673858e+03   0.000e+00   3.459e+03   1.426e-04        5
          60    8.683483e+03   0.000e+00   5.619e+02   4.284e-06        5
          70    8.683690e+03   0.000e+00   4.927e+02   1.183e-08        8
          78    8.683690e+03   0.000e+00   4.927e+02   9.036e-17       12
    
    EXIT: Primal feasible solution; terminate because the relative change in
          solution estimate < xtol.
    
    Final Statistics
    ----------------
    Final objective value               =   8.68368980794010e+03
    Final feasibility error (abs / rel) =   0.00e+00 / 0.00e+00
    Final optimality error  (abs / rel) =   4.93e+02 / 1.00e+00
    # of iterations                     =         78 
    # of CG iterations                  =        742 
    # of function evaluations           =        420
    # of gradient evaluations           =         79
    # of Hessian evaluations            =         76
    Total program time (secs)           =       8.49573 (     8.460 CPU time)
    Time spent in evaluations (secs)    =       8.47474
    
    ===============================================================================
    
      8.877926 seconds (398.54 M allocations: 6.096 GB, 15.25% gc time)


    ### Could not find a valid license.
        Your machine ID is e2-9d-cc-64-5f.
        Please contact licensing@artelys.com or your local distributor to obtain a license.
        If you already have a license, please execute `get_machine_ID -v` and send the output to support.





    VarianceComponentModels.VarianceComponentModel{Float64,2,Array{Float64,2},Array{Float64,2}}(0x5 Array{Float64,2},(
    5x5 Array{Float64,2}:
     0.31512   0.319157  0.250341  0.204466  0.24944 
     0.319157  0.329653  0.241834  0.233293  0.247452
     0.250341  0.241834  0.221271  0.113265  0.208544
     0.204466  0.233293  0.113265  0.241484  0.139501
     0.24944   0.247452  0.208544  0.139501  0.376927,
    
    5x5 Array{Float64,2}:
     0.688825  0.632998   0.657586    0.196702   0.528705
     0.632998  0.685078   0.56295     0.408814   0.507746
     0.657586  0.56295    0.775116   -0.0757512  0.501867
     0.196702  0.408814  -0.0757512   0.78627    0.191494
     0.528705  0.507746   0.501867    0.191494   0.756279),0x0 Array{Float64,2},Char[],Float64[],-Inf,Inf)



We then run the MM algorithm, starting from the Fisher scoring answer. MM finds an improved solution with objective value 8.955397e+03.


```julia
# trait59_model contains the fitted model by Fisher scoring now
@time mle_mm!(trait59_model, trait59_data; verbose=true)
trait59_model
```

    
         MM Algorithm
      Iter      Objective  
    --------  -------------
           0   8.683690e+03
           1   8.749335e+03
           2   8.769373e+03
           3   8.775609e+03
           4   8.777713e+03
           5   8.778568e+03
           6   8.779034e+03
           7   8.779373e+03
           8   8.779666e+03
           9   8.779940e+03
          10   8.780204e+03
          20   8.782575e+03
          30   8.784706e+03
          40   8.786833e+03
          50   8.789198e+03
          60   8.792077e+03
          70   8.795815e+03
          80   8.800822e+03
          90   8.807545e+03
         100   8.816372e+03
         110   8.827478e+03
         120   8.840646e+03
         130   8.855183e+03
         140   8.870054e+03
         150   8.884191e+03
         160   8.896801e+03
         170   8.907505e+03
         180   8.916279e+03
         190   8.923318e+03
         200   8.928908e+03
         210   8.933336e+03
         220   8.936854e+03
         230   8.939668e+03
         240   8.941938e+03
         250   8.943788e+03
         260   8.945309e+03
         270   8.946573e+03
         280   8.947632e+03
         290   8.948528e+03
         300   8.949292e+03
         310   8.949948e+03
         320   8.950516e+03
         330   8.951009e+03
         340   8.951441e+03
         350   8.951820e+03
         360   8.952155e+03
         370   8.952453e+03
         380   8.952717e+03
         390   8.952954e+03
         400   8.953165e+03
         410   8.953356e+03
         420   8.953527e+03
         430   8.953682e+03
         440   8.953823e+03
         450   8.953950e+03
         460   8.954066e+03
         470   8.954171e+03
         480   8.954267e+03
         490   8.954355e+03
         500   8.954435e+03
         510   8.954509e+03
         520   8.954576e+03
         530   8.954638e+03
         540   8.954695e+03
         550   8.954748e+03
         560   8.954796e+03
         570   8.954841e+03
         580   8.954882e+03
         590   8.954920e+03
         600   8.954955e+03
         610   8.954987e+03
         620   8.955017e+03
         630   8.955045e+03
         640   8.955071e+03
         650   8.955094e+03
         660   8.955117e+03
         670   8.955137e+03
         680   8.955156e+03
         690   8.955174e+03
         700   8.955190e+03
         710   8.955205e+03
         720   8.955219e+03
         730   8.955233e+03
         740   8.955245e+03
         750   8.955256e+03
         760   8.955267e+03
         770   8.955276e+03
         780   8.955286e+03
         790   8.955294e+03
         800   8.955302e+03
         810   8.955309e+03
         820   8.955316e+03
         830   8.955323e+03
         840   8.955329e+03
         850   8.955334e+03
         860   8.955339e+03
         870   8.955344e+03
         880   8.955349e+03
         890   8.955353e+03
         900   8.955357e+03
         910   8.955360e+03
         920   8.955364e+03
         930   8.955367e+03
         940   8.955370e+03
         950   8.955372e+03
         960   8.955375e+03
         970   8.955377e+03
         980   8.955380e+03
         990   8.955382e+03
        1000   8.955384e+03
        1010   8.955385e+03
        1020   8.955387e+03
        1030   8.955389e+03
        1040   8.955390e+03
        1050   8.955391e+03
        1060   8.955393e+03
        1070   8.955394e+03
        1080   8.955395e+03
        1090   8.955396e+03
        1100   8.955397e+03
    
     10.940231 seconds (375.19 M allocations: 9.267 GB, 19.34% gc time)





    VarianceComponentModels.VarianceComponentModel{Float64,2,Array{Float64,2},Array{Float64,2}}(0x5 Array{Float64,2},(
    5x5 Array{Float64,2}:
     0.261514  0.273061  0.22074    0.167275   0.216444
     0.273061  0.290848  0.219592   0.198638   0.218758
     0.22074   0.219592  0.207146   0.0954691  0.19478 
     0.167275  0.198638  0.0954691  0.207501   0.110312
     0.216444  0.218758  0.19478    0.110312   0.216043,
    
    5x5 Array{Float64,2}:
     0.735836  0.667979   0.685261    0.21787    0.535609
     0.667979  0.708913   0.582131    0.42248    0.492028
     0.685261  0.582131   0.789681   -0.0636001  0.484483
     0.21787   0.42248   -0.0636001   0.792537   0.187238
     0.535609  0.492028   0.484483    0.187238   0.781172),0x0 Array{Float64,2},Char[],Float64[],-Inf,Inf)



Do another run of MM algorithm from default starting point. It leads to a slightly better local optimum 8.957172e+03. Follow up anlaysis should use this result.


```julia
# default starting point
trait59_model = VarianceComponentModel(trait59_data)
@time _, _, _, Σcov, = mle_mm!(trait59_model, trait59_data; verbose=true)
trait59_model
```

    
         MM Algorithm
      Iter      Objective  
    --------  -------------
           0  -5.042149e+04
           1  -1.699687e+04
           2  -1.543968e+03
           3   5.143609e+03
           4   7.669359e+03
           5   8.489390e+03
           6   8.728893e+03
           7   8.796019e+03
           8   8.815880e+03
           9   8.823279e+03
          10   8.827419e+03
          20   8.854389e+03
          30   8.874943e+03
          40   8.890795e+03
          50   8.903062e+03
          60   8.912619e+03
          70   8.920130e+03
          80   8.926088e+03
          90   8.930861e+03
         100   8.934721e+03
         110   8.937872e+03
         120   8.940467e+03
         130   8.942622e+03
         140   8.944425e+03
         150   8.945946e+03
         160   8.947237e+03
         170   8.948339e+03
         180   8.949287e+03
         190   8.950106e+03
         200   8.950818e+03
         210   8.951439e+03
         220   8.951983e+03
         230   8.952462e+03
         240   8.952886e+03
         250   8.953262e+03
         260   8.953597e+03
         270   8.953896e+03
         280   8.954164e+03
         290   8.954404e+03
         300   8.954621e+03
         310   8.954817e+03
         320   8.954994e+03
         330   8.955154e+03
         340   8.955300e+03
         350   8.955433e+03
         360   8.955555e+03
         370   8.955666e+03
         380   8.955767e+03
         390   8.955860e+03
         400   8.955946e+03
         410   8.956025e+03
         420   8.956097e+03
         430   8.956164e+03
         440   8.956226e+03
         450   8.956283e+03
         460   8.956336e+03
         470   8.956385e+03
         480   8.956430e+03
         490   8.956473e+03
         500   8.956512e+03
         510   8.956549e+03
         520   8.956583e+03
         530   8.956615e+03
         540   8.956645e+03
         550   8.956673e+03
         560   8.956699e+03
         570   8.956723e+03
         580   8.956746e+03
         590   8.956768e+03
         600   8.956788e+03
         610   8.956807e+03
         620   8.956825e+03
         630   8.956842e+03
         640   8.956858e+03
         650   8.956873e+03
         660   8.956887e+03
         670   8.956900e+03
         680   8.956913e+03
         690   8.956925e+03
         700   8.956936e+03
         710   8.956947e+03
         720   8.956957e+03
         730   8.956967e+03
         740   8.956976e+03
         750   8.956985e+03
         760   8.956994e+03
         770   8.957001e+03
         780   8.957009e+03
         790   8.957016e+03
         800   8.957023e+03
         810   8.957030e+03
         820   8.957036e+03
         830   8.957042e+03
         840   8.957048e+03
         850   8.957053e+03
         860   8.957058e+03
         870   8.957064e+03
         880   8.957068e+03
         890   8.957073e+03
         900   8.957077e+03
         910   8.957082e+03
         920   8.957086e+03
         930   8.957090e+03
         940   8.957093e+03
         950   8.957097e+03
         960   8.957100e+03
         970   8.957104e+03
         980   8.957107e+03
         990   8.957110e+03
        1000   8.957113e+03
        1010   8.957116e+03
        1020   8.957119e+03
        1030   8.957121e+03
        1040   8.957124e+03
        1050   8.957126e+03
        1060   8.957129e+03
        1070   8.957131e+03
        1080   8.957133e+03
        1090   8.957135e+03
        1100   8.957138e+03
        1110   8.957140e+03
        1120   8.957141e+03
        1130   8.957143e+03
        1140   8.957145e+03
        1150   8.957147e+03
        1160   8.957149e+03
        1170   8.957150e+03
        1180   8.957152e+03
        1190   8.957153e+03
        1200   8.957155e+03
        1210   8.957156e+03
        1220   8.957158e+03
        1230   8.957159e+03
        1240   8.957160e+03
        1250   8.957161e+03
        1260   8.957163e+03
        1270   8.957164e+03
        1280   8.957165e+03
        1290   8.957166e+03
        1300   8.957167e+03
        1310   8.957168e+03
        1320   8.957169e+03
        1330   8.957170e+03
        1340   8.957171e+03
        1350   8.957172e+03
    
     13.052188 seconds (455.86 M allocations: 11.254 GB, 19.84% gc time)





    VarianceComponentModels.VarianceComponentModel{Float64,2,Array{Float64,2},Array{Float64,2}}(0x5 Array{Float64,2},(
    5x5 Array{Float64,2}:
     0.28408   0.282346  0.235651   0.163231   0.243528
     0.282346  0.287797  0.222888   0.189209   0.231229
     0.235651  0.222888  0.215833   0.0895747  0.212778
     0.163231  0.189209  0.0895747  0.199473   0.107541
     0.243528  0.231229  0.212778   0.107541   0.246379,
    
    5x5 Array{Float64,2}:
     0.714039  0.65898    0.67085     0.221737   0.509435
     0.65898   0.711746   0.578905    0.431453   0.480001
     0.67085   0.578905   0.781276   -0.0579601  0.467102
     0.221737  0.431453  -0.0579601   0.800138   0.189943
     0.509435  0.480001   0.467102    0.189943   0.751844),0x0 Array{Float64,2},Char[],Float64[],-Inf,Inf)



Heritability from 5-variate estimate and their standard errors.


```julia
h, hse = heritability(trait59_model.Σ, Σcov)
[h'; hse']
```




    2x5 Array{Float64,2}:
     0.284615   0.287928   0.216459   0.199551  0.246817 
     0.0773858  0.0769331  0.0836844  0.085051  0.0809202



## 13-trait analysis

Researchers would like to jointly analyze all 13 traits. Fisher scoring algorithm (with KNITRO backend) ends with objective value of -6.839422e+04.


```julia
# initialize model parameters
alltrait_model = VarianceComponentModel(cg10kdata_rotated)
# estimate variance components
@time mle_fs!(alltrait_model, cg10kdata_rotated; solver=:Knitro, verbose=true)
alltrait_model
```

    
    Knitro 10.1.0 STUDENT LICENSE (problem size limit = 300)
    
    =======================================
                Student License
           (NOT FOR COMMERCIAL USE)
             Artelys Knitro 10.1.0
    =======================================
    
    Knitro presolve eliminated 0 variables and 0 constraints.
    
    algorithm:            1
    The problem is identified as bound constrained only.
    Knitro changing bar_initpt from AUTO to 3.
    Knitro changing bar_murule from AUTO to 4.
    Knitro changing bar_penaltycons from AUTO to 1.
    Knitro changing bar_penaltyrule from AUTO to 2.
    Knitro changing bar_switchrule from AUTO to 1.
    Knitro changing linsolver from AUTO to 2.
    
    Problem Characteristics                    ( Presolved)
    -----------------------
    Objective goal:  Maximize
    Number of variables:                   182 (       182)
        bounded below:                      26 (        26)
        bounded above:                       0 (         0)
        bounded below and above:             0 (         0)
        fixed:                               0 (         0)
        free:                              156 (       156)
    Number of constraints:                   0 (         0)
        linear equalities:                   0 (         0)
        nonlinear equalities:                0 (         0)
        linear inequalities:                 0 (         0)
        nonlinear inequalities:              0 (         0)
        range:                               0 (         0)
    Number of nonzeros in Jacobian:          0 (         0)
    Number of nonzeros in Hessian:       16653 (     16653)
    
      Iter      Objective      FeasError   OptError    ||Step||    CGits 
    --------  --------------  ----------  ----------  ----------  -------
           0   -1.311337e+05   0.000e+00
          10   -7.616310e+04   0.000e+00   8.995e+02   4.820e-01        0
          20   -6.839422e+04   0.000e+00   2.109e+03   0.000e+00       38
    
    EXIT: Primal feasible solution; terminate because the relative change in
          solution estimate < xtol.
    
    Final Statistics
    ----------------
    Final objective value               =  -6.83942246370272e+04
    Final feasibility error (abs / rel) =   0.00e+00 / 0.00e+00
    Final optimality error  (abs / rel) =   2.11e+03 / 1.00e+00
    # of iterations                     =         21 
    # of CG iterations                  =        358 
    # of function evaluations           =        114
    # of gradient evaluations           =         22
    # of Hessian evaluations            =         20
    Total program time (secs)           =      15.45499 (    28.004 CPU time)
    Time spent in evaluations (secs)    =      15.41301
    
    ===============================================================================
    
     16.196635 seconds (734.69 M allocations: 11.184 GB, 18.44% gc time)


    ### Could not find a valid license.
        Your machine ID is e2-9d-cc-64-5f.
        Please contact licensing@artelys.com or your local distributor to obtain a license.
        If you already have a license, please execute `get_machine_ID -v` and send the output to support.





    VarianceComponentModels.VarianceComponentModel{Float64,2,Array{Float64,2},Array{Float64,2}}(0x13 Array{Float64,2},(
    13x13 Array{Float64,2}:
      0.314331    0.132105     -0.0353743   …  -0.136065    -0.105679  
      0.132105    0.139835      0.140532       -0.0173756   -0.00454843
     -0.0353743   0.140532      0.356882        0.0759456    0.0921647 
      0.343477    0.054447     -0.20584        -0.178914    -0.150596  
     -0.129413    0.0162518     0.189194        0.074017     0.0695955 
     -0.113535    0.00983593    0.160915    …   0.0670008    0.0576629 
     -0.103928    0.012809      0.16756         0.0753421    0.0715201 
     -0.0549071   0.000400283   0.0502762       0.00854523   0.00326281
     -0.205264   -0.0387588     0.153961        0.0858418    0.0789957 
     -0.147011   -0.0856788    -0.064831        0.12759      0.0730245 
     -0.155356   -0.0717934    -0.00878633  …   0.0945906    0.0510817 
     -0.136065   -0.0173756     0.0759456       0.357044     0.236415  
     -0.105679   -0.00454843    0.0921647       0.236415     0.24394   ,
    
    13x13 Array{Float64,2}:
      0.701526    0.640432    -0.0964342  …  -0.352486   -0.0542532  -0.125562 
      0.640432    0.884372     0.107444      -0.340323   -0.0140205   0.0218768
     -0.0964342   0.107444     0.649201      -0.127019    0.0354407   0.0121441
      0.496518    0.241287    -0.254125      -0.219282   -0.0580118  -0.19055  
     -0.282266   -0.0653499    0.327741      -0.0412117   0.0425152   0.0545268
     -0.259093   -0.0348929    0.297623   …  -0.0583661   0.0376783   0.0533446
     -0.249      -0.0489288    0.349755      -0.0283019   0.0338332   0.0434008
     -0.105542    0.00459972   0.0387758     -0.0633714   0.0178299   0.0319088
     -0.32499    -0.126343     0.252081       0.13956     0.0987083   0.102378 
     -0.266693   -0.308904    -0.160801       0.422532    0.0144221   0.0601095
     -0.352486   -0.340323    -0.127019   …   0.788049    0.107235    0.146221 
     -0.0542532  -0.0140205    0.0354407      0.107235    0.645093    0.405515 
     -0.125562    0.0218768    0.0121441      0.146221    0.405515    0.755077 ),0x0 Array{Float64,2},Char[],Float64[],-Inf,Inf)



Re-run using MM algorithm, which locates a better mode with objective value -4.435632e+04. Follow up anlaysis should use this result.


```julia
# initialize model parameters
alltrait_model = VarianceComponentModel(cg10kdata_rotated)
# estimate variance components
@time _, _, _, Σcov, = mle_mm!(alltrait_model, cg10kdata_rotated; verbose=true)
alltrait_model
```

    
         MM Algorithm
      Iter      Objective  
    --------  -------------
           0  -1.311337e+05
           1  -8.002195e+04
           2  -5.807051e+04
           3  -4.926234e+04
           4  -4.611182e+04
           5  -4.511727e+04
           6  -4.482798e+04
           7  -4.474410e+04
           8  -4.471610e+04
           9  -4.470285e+04
          10  -4.469355e+04
          20  -4.462331e+04
          30  -4.456960e+04
          40  -4.452834e+04
          50  -4.449652e+04
          60  -4.447178e+04
          70  -4.445237e+04
          80  -4.443699e+04
          90  -4.442467e+04
         100  -4.441470e+04
         110  -4.440656e+04
         120  -4.439985e+04
         130  -4.439427e+04
         140  -4.438959e+04
         150  -4.438564e+04
         160  -4.438229e+04
         170  -4.437941e+04
         180  -4.437694e+04
         190  -4.437480e+04
         200  -4.437294e+04
         210  -4.437131e+04
         220  -4.436989e+04
         230  -4.436863e+04
         240  -4.436751e+04
         250  -4.436652e+04
         260  -4.436564e+04
         270  -4.436485e+04
         280  -4.436414e+04
         290  -4.436351e+04
         300  -4.436293e+04
         310  -4.436242e+04
         320  -4.436195e+04
         330  -4.436152e+04
         340  -4.436113e+04
         350  -4.436078e+04
         360  -4.436046e+04
         370  -4.436016e+04
         380  -4.435989e+04
         390  -4.435965e+04
         400  -4.435942e+04
         410  -4.435921e+04
         420  -4.435902e+04
         430  -4.435884e+04
         440  -4.435867e+04
         450  -4.435852e+04
         460  -4.435838e+04
         470  -4.435825e+04
         480  -4.435813e+04
         490  -4.435802e+04
         500  -4.435791e+04
         510  -4.435781e+04
         520  -4.435772e+04
         530  -4.435764e+04
         540  -4.435756e+04
         550  -4.435748e+04
         560  -4.435741e+04
         570  -4.435735e+04
         580  -4.435729e+04
         590  -4.435723e+04
         600  -4.435718e+04
         610  -4.435713e+04
         620  -4.435708e+04
         630  -4.435704e+04
         640  -4.435700e+04
         650  -4.435696e+04
         660  -4.435692e+04
         670  -4.435688e+04
         680  -4.435685e+04
         690  -4.435682e+04
         700  -4.435679e+04
         710  -4.435676e+04
         720  -4.435674e+04
         730  -4.435671e+04
         740  -4.435669e+04
         750  -4.435667e+04
         760  -4.435665e+04
         770  -4.435663e+04
         780  -4.435661e+04
         790  -4.435659e+04
         800  -4.435657e+04
         810  -4.435656e+04
         820  -4.435654e+04
         830  -4.435653e+04
         840  -4.435651e+04
         850  -4.435650e+04
         860  -4.435649e+04
         870  -4.435648e+04
         880  -4.435647e+04
         890  -4.435646e+04
         900  -4.435645e+04
         910  -4.435644e+04
         920  -4.435643e+04
         930  -4.435642e+04
         940  -4.435641e+04
         950  -4.435640e+04
         960  -4.435639e+04
         970  -4.435639e+04
         980  -4.435638e+04
         990  -4.435637e+04
        1000  -4.435637e+04
        1010  -4.435636e+04
        1020  -4.435635e+04
        1030  -4.435635e+04
        1040  -4.435634e+04
        1050  -4.435634e+04
        1060  -4.435633e+04
        1070  -4.435633e+04
        1080  -4.435632e+04
    
     27.474548 seconds (976.37 M allocations: 23.795 GB, 18.43% gc time)





    VarianceComponentModels.VarianceComponentModel{Float64,2,Array{Float64,2},Array{Float64,2}}(0x13 Array{Float64,2},(
    13x13 Array{Float64,2}:
      0.273498    0.192141    -0.0207392   0.231615   …  -0.128643   -0.098307  
      0.192141    0.219573     0.134389    0.0800317     -0.0687355  -0.0433724 
     -0.0207392   0.134389     0.329149   -0.158086       0.0717258   0.097381  
      0.231615    0.0800317   -0.158086    0.277921      -0.129316   -0.104571  
     -0.149294   -0.0130248    0.186275   -0.219746       0.0673901   0.0798606 
     -0.131878   -0.00396265   0.169037   -0.204038   …   0.0539454   0.0662375 
     -0.14514    -0.0332988    0.168384   -0.190066       0.0778299   0.0867255 
     -0.0299641   0.0372326    0.0617453  -0.0918129     -0.0120473  -0.00309877
     -0.205216   -0.0839698    0.137149   -0.232933       0.0895106   0.0971193 
     -0.100475   -0.114933    -0.088079   -0.0435633      0.046928   -0.0048387 
     -0.133733   -0.109897    -0.0247458  -0.0985453  …   0.0587572   0.0114353 
     -0.128643   -0.0687355    0.0717258  -0.129316       0.117185    0.0899824 
     -0.098307   -0.0433724    0.097381   -0.104571       0.0899824   0.106354  ,
    
    13x13 Array{Float64,2}:
      0.723441    0.568135    -0.113563     0.590598   …  -0.0586259  -0.12469   
      0.568135    0.77999      0.109287     0.215784       0.0236098   0.0464835 
     -0.113563    0.109287     0.669754    -0.299757       0.0467158   0.00601682
      0.590598    0.215784    -0.299757     0.718142      -0.0951686  -0.218709  
     -0.252371   -0.0353068    0.334628    -0.372904       0.0472511   0.0435499 
     -0.228865   -0.0203083    0.295345    -0.343471   …   0.0511838   0.0482913 
     -0.193033    0.00191887   0.345731    -0.319          0.0327529   0.0272593 
     -0.129704   -0.0365696    0.0400358   -0.156082       0.0427586   0.0451036 
     -0.30716    -0.0823855    0.2673      -0.410123       0.102842    0.0946414 
     -0.303001   -0.28147     -0.13149     -0.218          0.0947723   0.142937  
     -0.364401   -0.30413     -0.107477    -0.270766   …   0.143809    0.187604  
     -0.0586259   0.0236098    0.0467158   -0.0951686      0.881707    0.551818  
     -0.12469     0.0464835    0.00601682  -0.218709       0.551818    0.893023  ),0x0 Array{Float64,2},Char[],Float64[],-Inf,Inf)



Heritability estimate from 13-variate analysis.


```julia
h, hse = heritability(alltrait_model.Σ, Σcov)
[h'; hse']
```




    2x13 Array{Float64,2}:
     0.274338   0.219669   0.329511  0.279019   …  0.172946  0.117315   0.10642  
     0.0778364  0.0818365  0.072716  0.0775316     0.087259  0.0902115  0.0903969



## Save analysis results


```julia
using JLD
@save "copd.jld"
whos()
```

    INFO: Recompiling stale cache file /Users/huazhou/.julia/lib/v0.4/JLD.ji for module JLD.


                     #334#wsession    280 bytes  JLD.JldWriteSession
                        ArrayViews    188 KB     Module
                              Base  38129 KB     Module
                           BinDeps    208 KB     Module
                             Blosc     37 KB     Module
                        ColorTypes    313 KB     Module
                            Colors    747 KB     Module
                            Compat    344 KB     Module
                             Conda     65 KB     Module
                              Core   7200 KB     Module
                        DataArrays    762 KB     Module
                        DataFrames   1807 KB     Module
                            Docile    415 KB     Module
                            FileIO    536 KB     Module
                 FixedPointNumbers     33 KB     Module
                   FixedSizeArrays    157 KB     Module


    WARNING: both DataArrays and StatsBase export "autocor"; uses of it in module DataFrames must be qualified
    WARNING: both DataArrays and StatsBase export "inverse_rle"; uses of it in module DataFrames must be qualified
    WARNING: both DataArrays and StatsBase export "rle"; uses of it in module DataFrames must be qualified


                              GZip    791 KB     Module
                              HDF5   3336 KB     Module
                            IJulia 3468156 KB     Module
                    IPythonDisplay     35 KB     Module
                             Ipopt     50 KB     Module
                  IterativeSolvers    486 KB     Module
                               JLD   1364 KB     Module
                              JSON    240 KB     Module
                            KNITRO    320 KB     Module
                      LaTeXStrings   3115 bytes  Module
                        MacroTools    124 KB     Module
                              Main 3527139 KB     Module
                      MathProgBase   1393 KB     Module
                          Measures     15 KB     Module
                            Nettle     58 KB     Module
                             Plots   2975 KB     Module
                            PyCall   1002 KB     Module
                            PyPlot   1278 KB     Module
                       RecipesBase    193 KB     Module
                          Reexport   3658 bytes  Module
                               SHA     50 KB     Module
                         SnpArrays    437 KB     Module
                 SortingAlgorithms     40 KB     Module
                         StatsBase    783 KB     Module
                         StatsFuns    286 KB     Module
                         URIParser    103 KB     Module
           VarianceComponentModels    642 KB     Module
                                 Y    677 KB     6670x13 Array{Float64,2}
                               ZMQ     81 KB     Module
                                 _   2720 bytes  Tuple{Array{Float64,2},Array{Float…
                    alltrait_model   2792 bytes  VarianceComponentModels.VarianceCo…
                             cg10k 1027303 KB     6670x630860 SnpArrays.SnpArray{2}
                       cg10k_trait    978 KB     6670×15 DataFrames.DataFrame
                         cg10kdata 695816 KB     VarianceComponentModels.VarianceCo…
                 cg10kdata_rotated    729 KB     VarianceComponentModels.TwoVarComp…
                                 h    104 bytes  13-element Array{Float64,1}
                               hST    104 bytes  13-element Array{Float64,1}
                            hST_se    104 bytes  13-element Array{Float64,1}
                               hse    104 bytes  13-element Array{Float64,1}
                               maf   4928 KB     630860-element Array{Float64,1}
                      minor_allele     77 KB     630860-element BitArray{1}
                missings_by_person     52 KB     6670-element Array{Int64,1}
                   missings_by_snp   4928 KB     630860-element Array{Int64,1}
                            people      8 bytes  Int64
                              snps      8 bytes  Int64
                      trait59_data    312 KB     VarianceComponentModels.TwoVarComp…
                     trait59_model    488 bytes  VarianceComponentModels.VarianceCo…
                          traitidx     16 bytes  5-element UnitRange{Int64}
                                Σa   3848 bytes  13x13 Array{Array{Float64,2},2}
                              Σcov    892 KB     338x338 Array{Float64,2}
                                Σe   3848 bytes  13x13 Array{Array{Float64,2},2}
                              Φgrm 347569 KB     6670x6670 Array{Float64,2}
                               σ2a    104 bytes  13-element Array{Float64,1}
                               σ2e    104 bytes  13-element Array{Float64,1}



```julia

```