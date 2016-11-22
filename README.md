# stacked_tensor

In several data processing situations (e.g. experimental data, convolution neural networks) there is a need to manipulate tensors in a way to represent them in reduced dimensions by stacking two or more dimensions. This script aims at simplifying the process by expressing all the dimensions in the stacked condition and manipulate them

In 2013 I released this in Matlab on [https://it.mathworks.com/matlabcentral/fileexchange/42145-stacked-matrix-permutation]|FileExchange] and here I am adding the numpy version.

## Problem
Input: tensor with flattened content in each dimension expressed as ((A,B,C),(D,E),(F,G,H)) where inside each dimension we express it using row major.

Output: tensor with the sub-dimensios reordered abritrarely e.g. ((F,B,D),(A,C),(H,E))

Formally we have real tensor dimensions (a_i,s_i) and sub-dimensions (a_ij,s_ij). Meaning that the real tensor is:

  a_i = join (a_ij for j in i)
  s_i = sum (s_ij)

In a tensor two elements can be used for generically expressing the behavior of an index: stride. For the case (a1,s1),(a2,s2),(a3,s3) and row major we compute right to left the strides: (s3s2,s3,1) while for column major we compute left to right: (1,s1,s2s1).

The intermediate step of the approach is the expanded tensor in which subdimensions are full dimensions. If we still assume everything is row major then the expansion goes all left to right:

packed → expanded → reorder → packed

Packed: ((A,B,C),(D,E),(F,G,H))
Unpacked: (A,B,C,D,E,F,G,H)
Requested Packed: ((F,B,D),(A,C),(H,E))
Unpacked: (F,B,D,A,C,H,E)

## Approach

in Matlab (column major) we unpack the tensor using reshape into a regular tensor, reorder the dimensions and the reshape them in the final ones. Unpacking/packing is done via reshape, while permutation is done with permute. Matlab is column major so any unpacked list needs to be reversed before usage in the reshape/permute.

# Example

In the example below we create a matrix with the original index as the value and then permute it:

	a = genmatrix([2,3,4])
	da,db,dc = makeflatsubdims(a,("a","b","c"))
	af = (da,db,dc)
	rf = (da,(db,dc))
	r = stackedpermute(a,af,rf,verbose=True)

# CNN Example
A practical example in CNN is the following. Assuming that we are dealing with a timeseries of T samples made of F features (position,velocity) in 3 dimensions (xyz). This means that the input can be assumed to be (T,F,C). Then, for perforning convolution we need to reduce to a 2 dimension matrix along several approaches:

* F|T,C that is we have F separate convolutions of matrices T by C
* T,FC  that is we have a matrix with time on rows, and stacked F and C on columns
* T,CF  as above witht C and F flipped


