> [!WARNING]
> Work in progress specification.

# ZIPFS Specification

## Introduction

ZIPFS is a [dictionary-based](https://en.wikipedia.org/wiki/Dictionary_coder) method for compression and decompression of files, lossless, and with support of the [IPFS network](https://ipfs.tech/). It is not an algorithm itself but may rely on several underlying algorithms.

The compression consists in finding the most frequent arbitrary size segments (i.e. substrings) of a file and giving them the [Huffman coding](https://en.wikipedia.org/wiki/Huffman_coding), composing the dictionary. Due to the advantages of a smaller dictionary size and the simplification of the compression and decompression algorithm, the [Canonical Huffman code](https://en.wikipedia.org/wiki/Canonical_Huffman_code) is used. 

Large dictionaries trained for specific file types can be built and stored in the IPFS network, hence, ZIPFS qualifies as a static dictionary method. The algorithm used to build the dictionary is not fixed by the specification and can be chosen from any available option.

The decompression process consists of download the respective dictionary from the IPFS network and decode the compressed file, which is composed of Huffman codes. A compressed file does not carry any dictionary since it is already stored in the IPFS network. 

This approach has the following advantages:

1. Dictionaries should not be trained using private data, they should be built from public datasets. It’s possible to store large, comprehensive shared dictionaries tailored to different data types. I believe that compression ratios can be significantly improved compared to local dictionary methods.
2. Dictionary downloads tend to be fast and parallelizable. Large dictionaries can be split into smaller chunks, allowing for faster, efficient, and more scalable downloads of large compressed files, especially compared to centralized sources.
3. Dictionaries contain only meaningless data. The actual information remains stored locally, making it completely secure. This is particularly useful in scenarios involving private data, regulatory compliance, or even as an alternative to encryption in certain use cases.

## Dictionary

A dictionary is a data structure composed of metadata and a main section, which is a list of file segments ordered from the shortest Huffman code to the longest. The Huffman codes themselves are omitted, since their values are already known.

Dictionaries are the public portion of compressed files and are designed to be stored on the IPFS network. The [InterPlanetary Linked Data (IPLD)](https://ipld.io/) standard is used to specify and construct such structures, that should be encoded using the codec [DAG-CBOR](https://ipld.io/docs/codecs/known/dag-cbor/).

Two types of dictionaries are specified: the first is the single-block dictionary, designed to store small dictionaries that must fit entirely within a single block. The second is the multi-block dictionary, which can split the dictionary across multiple blocks, intended for large dictionaries and optimized for download and decompression.

### Single-block dictionary

Single-block dictionary have the layout:

```
type Dictionary struct {
	Name                 String
	Description optional String
	Segments             [Bytes]
}
```
- Name is the name of the dictionary.
- Description is an optional field that may contain any relevant metadata provided by the dictionary’s creator, such as the algorithm used, the type of target file, the version, or any other relevant information.
- Segments is a List of Lists of bytes, where each element of the outer list corresponds to a file segment. This field must be ordered from the segment corresponding to the shortest Huffman code to the one corresponding to the longest.


### Multi-block dictionary

TODO

### Dictionary training

TODO

## Compression and decompression

Compression consists of taking an original file, provided by the user, as input and producing a compressed file as output. The input file may be any file in any format, and its specification is outside the scope of this document. The output file, or compressed file, however, follows a standard that must be defined by the ZIPFS method. The decompression process is the inverse of compression, it takes the compressed file as input and produces the original file as output.

### Compressed file

A compressed file essentially consists of a sequence of Huffman codes that represent the corresponding segments found in the dictionary. The dictionary is stored on the IPFS network and must be referenced by its CID. The file has the following layout:

```
type CompressedFile struct {
	Name       String
	Dictionary CID
	Data       [Byte]
}
```
- Name is the name of the original file.
- Dictionary is the dictionary containing the file segments, represented by a CID that addresses it on the IPFS network.
- Data are the Huffman codes that make up the file. The codes are packed (concatenated) and may receive final padding to complete a byte array.

### Compression

Let a file of $n$ bytes be denoted as $F = f_0 f_1 \ldots f_{n-1}$ and the dictionary with $m$ segments as $D = (d_0, d_1, \ldots, d_{m-1})$, where the order of the segments matters, since earlier segments receive shorter Huffman codes. The compression process consists of replacing occurrences of segment $d_i$ in the file $F$ with its corresponding Huffman code. In the case of ambiguity between segments of different lengths, the segment with the shorter Huffman code, or the one that appears earlier in $D$, is always chosen.

For example, if the file $F$ consists of an english text and contains the segment `alte`, and the dictionary $D$ has two ambiguous segments $f_i = a$ and $f_j = alt$ with $i < j$, then the segment `a` in the file is replaced by its Huffman code, since this segment has a shorter code because $i < j$. The remaining substring `lte` in $F$ must then be replaced by other codes according to the dictionary.

Below is pseudocode that implements a compression algorithm as described. Note that the goal of this algorithm is not efficiency, but rather to illustrate the functioning of the compression process.

```
INPUT: file F and dictionary D
OUTPUT: compressed file C
START
	FOR i IN LENGTH(D)
		FOR j = 0, j < LENGTH(F), j++
			IF isSameSegment(D,F,i,j)
				C[j] = 2^i-1
				j = j + LENGTH(D[i]) - 1
    		END IF
		END FOR
	END FOR
END
```
The auxiliary function *isSameSegment*:
```
FUNCTION isSameSegment(D,F,i,j):
	FOR z IN LENGTH(D[i])
		IF D[i][z] != F[j][z]
			RETURN false
		END IF
	END FOR
	RETURN true
END FUNCTION
```

## License

ZIPFS Specification is marked [CC0 1.0 Universal](./LICENSE).