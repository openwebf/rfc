# The WBC1 file format

## Abstract

This document defines the WBC1 (WebF Bytecode Format Version 1) file format, a file format designed for efficient transmission and execution of JavaScript in WebF applications. It specifies the structure, encoding, and metadata conventions for the WBC1 file format.

## 1. Introduction

### 1.1 Background

In Kraken, there is a similar file format called kbc1, which is comprised entriely by QuickJS bytecodes. However, due to lack of validation and compression mechanisms for kbc1 file, WebF cannot verify their integrity, potentially cause crashes or erros if the kbc1 file is not intact. Addtionally, without any compression, the raw kbc1 files are much larger than their corresponding JavaScript files, leading to increasd download and load times.

### 1.2. Purpose

The WBC1 file format is designed to address these issues by including a file checksum for integrity verifycation and compression the QuickJS bytecode binary using a high-performance lossless data compress algorithm.

### 1.3. Requirements

The WBC1 file is new file format needs to be support by newer WebF versions.

## 2. WBC1 File Structure

WBC1 files are organized into a series of chunks, each serving a specific purpose and containing different types of data. Here's an overview of the primary sections or chunks in a WBC1 file:

1. Signature (9 bytes): The file starts with an 9-byte signature that identifies it as a PNG file. 
2. The WBHD chunk: The first and most important chunk, containing essential properties compression method and compile level of the QuickJS bytecode.
3. The Quickjs bytecode data: Contains the compressed QuickJS bytecode data. 
4. The End chunk: Marks the end of the WBC1 file. It has no data field and is always the last chunk in the file.

### 2.1. Signature

The WBC1 file signature is a fixed sequence of 9 bytes:

0x89 0x57 0x42 0x43 0x31 0x0D 0x0A 0x1A 0x0A

This can be broken down as follows:

1. 0x89 - A high-bit set first byte to prevent transmission errors in some systems.
2. 0x57 0x42 0x43 0x31 - The ASCII values for the letters 'WBC1'.
3. 0D 0A - A CRLF (carriage return and line feed) sequence to detect corruption in some systems.
4. 1A - A control character (EOF) to help detect truncated files.
5. 0A - A final line feed character.

### 2.2. Header

The WBHD (WBC Header) chunk containing vital information about the file's properties. The WBHD has a specific structure consisting of:

1. Length (4 bytes): The length of data field in bytes, which is 17 bytes for WBHD.
2. Chunk type (4 bytes): ASCII value for the letter `WBHD` (0x57 0x42 0x48 0x44 in hexadecimal)
3. Chunk data: This part contains the actual wbc1 properties, broken down as follows:
   1. Compression method (1 bytes): Specifies the compression method used. Currently, only the value 0 is allowed, representing the LZ4 deflate/inflate compression method.
   2. QuickJS compile level (1 bytes): Specifies the compile level when produce the QuickJS bytecodes, different compile level would lead to different bytecode size and optimization level. Default to 0.
   3. QuickJS bytecode version (1 bytes): The QuickJS bytecode evaluation runs in compact mode by default, supporting all versions of both KBC1 and WBC1. However, compact mode may reduce the performance of evaluation. If this WBC1 is compiled by a newer version of QuickJS, the evaluator can use more efficient way to evaluate the bytecode and achieve higher performance. Default to 0.
   4. Additional data zone (3 bytes): preallocated space for other usage in the future.
4. CRC-32 (4 bytes): A 32-bit cyclic redundancy check (CRC) value, calculated from the header, to verify the integrity of the this chunk.

### 2.3 Body

Contains the compressed QuickJS bytecode data. 

The Body chunk has a specific structure consisting of:

1. Length (4 bytes): The length of data fields in bytes. This value varies depending on the size of the QuickJS bytecode size contained on the chunk.
2. Chunk Type (4 bytes): ASCII value for the letter `WBDY` (0x57 0x42 0x44 0x59 in hexadecimal)
3. Chunk data: this section contains of compressed QuickJS bytecode data. WBC1 use LZ4 lossless data compression method.
4. CRC-32 (4 bytes):  A 32-bit cyclic redundancy check (CRC) value, calculated from the body, to verify the integrity of the this chunk.

### 2.4 The End Chunk

It is always the last chunk in the file and has a specific structure:

1. Length (4 bytes): The length of the data field in bytes. 
2. Chunk type (4 bytes): The ASCII values for the letters 'WEND' (0x57 0x45 0x4e 0x44 in hexadecimal).

## 3. A example of WBC1 file

```
      M: Compression Method

      V: QuickJS ByteCode Version

      L: QuickJS Compile Level

      A: Addtional Data Zone

┌────┬────┬────┬────┬────┬────┬────┬────┬────┐
│0x98│0x57│0x42│0x43│0x31│0x0D│0x0A│0x1A│0x0A│  Signature
├────┴────┴────┴────┼────┴────┴────┴────┼────┤
│        Length     │    CHUNK_TYPE     │ M  │
├────┬────┬─────────┴────┬──────────────┴────┤  Header
│ L  │ V  │      A       │   CRC32 CHECKSUM  │
├────┴────┴─────────┬────┴──────────────┬────┤
│        Length     │    CHUNK_TYPE     │    │
├───────────────────┴───────────────────┘    │
│                                            │
│                                            │
│                                            │
│                                            │
│                                            │  BODY
│              QuickJS ByteCode BODY         │
│                                            │
│                                            │
│                        ┌───────────────────┤
│                        │   CRC32 CHECKSUM  │
├───────────────────┬────┴─────────────┬─────┘
│      Length       │    CHUNK_TYPE    │        END
└───────────────────┴──────────────────┘
```

## Compatibility with older version of WebF/Kraken

WBC1 is a new type of file that older version of WebF/Kraken cannot regonize and parse. Only versions of WebF released after the first supported version can accept this file.

## Tools and infrastructure

The WBC1 generator will locate at newer version of https://github.com/openwebf/node-qjsc

The newer [webf-cli](https://github.com/openwebf/cli) will ship a new command to compile a javascript file into a wbc1 file.

```
webf qjsc demo.js --pluginName "demo://" --wbc1 ./demo.wbc1
```

