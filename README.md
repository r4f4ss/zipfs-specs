> [!WARNING]
> Work in progress specification.

# ZIPFS Specification

## Introduction

ZIPFS is a [dictionary-based](https://en.wikipedia.org/wiki/Dictionary_coder) method for compression and decompression of files, lossless, and with support of the [IPFS network](https://ipfs.tech/). It is not an algorithm itself but may rely on several underlying algorithms.

The compression consists in finding the most frequent codes in a file and giving them the [Huffman coding](https://en.wikipedia.org/wiki/Huffman_coding), composing the dictionary. Large dictionaries trained for specific file types can be built and stored in the IPFS network, hence, ZIPFS qualifies as a static dictionary method. The algorithm used to build the dictionary is not fixed by the specification and can be chosen from any available option.

The decompression process consists of download the respective dictionary from the IPFS network and decode the compressed file, which is composed of Huffman codes. A compressed file does not carry any dictionary since it is already stored in the IPFS network. 

This approach has the following advantages:

1. Dictionaries should not be trained using private data, they should be built from public datasets. It’s possible to store large, comprehensive shared dictionaries tailored to different data types. I believe that compression ratios can be significantly improved compared to local dictionary methods.
2. Dictionary downloads tend to be fast and parallelizable. Large dictionaries can be split into smaller chunks, allowing for faster, efficient, and more scalable downloads of large compressed files, especially compared to centralized sources.
3. Dictionaries contain only meaningless data. The actual information remains stored locally, making it completely secure. This is particularly useful in scenarios involving private data, regulatory compliance, or even as an alternative to encryption in certain use cases.