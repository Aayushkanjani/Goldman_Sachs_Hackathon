# Goldman Sachs Hackathon

This document outlines three problems from GSIH'25.

---

## Problem 1: Bit Happens 

This problem involves solving a three-part digital forensics problem by decoding hidden patterns in various file formats.

### Subtask 1: Bitmap Steganography
* **Problem**: Decode a Base64 encoded BMP file to extract a hidden number from an `ABC{...}` pattern.
* **Approach**: The process involves decoding the Base64 string into a BMP file structure. The `PixelDataOffset` is read from the `BITMAPFILEHEADER` to locate the image payload. The Least Significant Bit (LSB) is extracted from the blue channel of each pixel. These bits are reassembled into bytes to form a message, from which the numeric pattern is extracted.

| Block | Field | Width | Description |
| :--- | :--- | :--- | :--- |
| **BITMAPFILEHEADER** | `FileType` | 2 bytes | ASCII string, must be 'BM'. |
| (Width: 14 bytes)  | `FileSize` | 4 bytes | Entire file size in bytes. |
| | `Reserved` | 2 bytes | Unused, initialized to 0. |
| | `Reserved` | 2 bytes | Unused, initialized to 0. |
| | `PixelDataOffset` | 4 bytes | Offset to the start of the pixel data (payload). |

### Subtask 2: PCAP Pattern Search
* **Problem**: Decode a Base64 PCAP blob, find a potentially large number within an `ABC{...}` pattern, and compute `(n % 10007) + 3`. If no pattern is found, the result is 0.
* **Approach**: The decoded PCAP data is treated as a single text blob. A regular expression is used to find the first occurrence of the `ABC{...}` pattern. The extracted number is handled using Java's `BigInteger` to support its large size , and the final value is computed.

### Subtask 3: MysticLang VM Simulation
* **Problem**: Simulate a custom 8-bit virtual machine to retrieve a byte from a specific memory address after the program halts.
* **Approach**: The Base64 hex bytecode is decoded and parsed into opcodes. A VM is initialized with 256 bytes of memory and 16 registers, all set to zero. The simulator fetches and executes instructions until the `HALT` opcode (0xFF) is encountered. The value at the requested memory address is then returned.

### Technology Selection: Java 
* **Reasoning**: Java was chosen for its strong byte-level control and endianness handling via `ByteBuffer` , as well as its high performance for executing loops over millions of VM operations.

### Limitations & Improvements
1.  **No IP/TCP/UDP Reassembly**:
    * **Current Fix**: The solution applies regex directly on the raw, reassembled PCAP blob decoded as ISO-8859-1 text.
    * **Future Fix**: Integrate a proper packet capture library (like jNetPcap) to reassemble packet fragments before searching.
2.  **VM Stack Pointer Underflow**:
    * **Current Fix**: The stack pointer (`%r15`) starts at 0 and underflow wraps around due to `& 0xFF` masking.
    * **Future Fix**: Initialize `%r15` to `0xFF` and add boundary checks to handle stack overflow/underflow explicitly.
3.  **Inefficient BMP LSB Scanning**:
    * **Current Fix**: The entire bitstream from the BMP is collected before the regex search begins.
    * **Future Fix**: Implement an early exit to stop bit extraction as soon as the `ABC{...}` pattern is successfully matched.

---

## Problem 2: Barcode Parsing - Code 39 Decoder

This problem decodes a Code 39 barcode from an image.

### Approach
The image processing pipeline involves several stages to handle real-world constraints like poor lighting and skewed angles.

1.  **Rotation Handling**: The input image is processed to correct for minor tilts.
2.  **Grayscale & Thresholding**: The image is converted to grayscale, and Otsu's thresholding algorithm is applied to create a binary (black and white) image.
3.  **Scanline Processing**: A few rows of pixels near the image's center are scanned to find the barcode pattern.
4.  **Width Encoding**: The widths of the black and white bars are measured and classified as narrow (N) or wide (W).
5.  **Bidirectional Decoding**: The pattern is decoded into characters. The scan can be done in both directions to handle images that may be upside down.

### Example Character Decoding
| Widths (9) | Pattern (N/W) | Char |
| :--- | :--- | :--- |
| [3,1,2,1,3,4,5,2,4] | NNNNNWWNW | G |
| [3,1,4,1,3,2,5,4,2] | NNWNNNWWN | S |
| [3,1,4,5,3,2,2,1,4] | NNWWNNNNW | 2 |
| [3,1,3,5,4,2,4,1,2] | NNNWWNWNN | 0 |
| [3,1,4,5,3,2,2,1,4] | NNWWNNNNW | 2 |
| [5,1,2,5,4,2,2,1,3] | WNNWWNNNN | 5 |


### Technology Selection: Python 
* **Reasoning**: Python was chosen for its fast prototyping capabilities and powerful imaging libraries like PIL and NumPy, which are ideal for iterative, logic-heavy pixel processing.

### Limitations & Improvements
1.  **Extreme Blur/Skew**:
    * **Current Fix**: Otsu's thresholding provides some robustness.
    * **Future Fix**: Implement deblurring filters or morphological operations to enhance edges.
2.  **Significant Tilt (> ±5°)**:
    * **Current Fix**: The current implementation handles only small-angle rotations.
    * **Future Fix**: Use a Hough-line transform to detect the barcode's angle and perform a full deskew.
3.  **Barcode Position and Size**:
    * **Current Fix**: The algorithm only checks a few rows around the center of the image.
    * **Future Fix**: Implement a sliding-window search with early pruning to find barcodes anywhere in the image.

---

## Problem 3: Automated Intersection Detection in 2D Line Graphs

This problem counts the number of line-line intersections in a 512x512 JPEG image of a 2D graph, excluding the graph's axes.

### Approach
The detection method is based on a multi-scale windowing heuristic.

1.  **Preprocessing**: The input image is converted to grayscale and then binarized.
2.  **Intersection Detection**: The algorithm slides `3x3`, `5x5`, and `7x7` windows across every pixel of the binary image.
3.  **Heuristic Validation**: A point is considered an intersection only if it satisfies a set of pixel density rules across the different window sizes. For example, a `7x7` window must not contain too many pixels (`<=30`) to reject thick blobs and axes. This confirms a true crossing of thin lines.
4.  **Merging**: Detections that are within a 6-pixel radius of each other are merged to prevent counting a single intersection multiple time.

### Assumptions
* Axes are thicker than data lines (>= 3px).
* Valid intersection points do not occur within a 6px radius of each other.
* L-shaped patterns are not considered intersections.

### Limitations & Improvements
1. **Heuristic Threshold Sensitivity**:
    * **Current Fix**: Thresholds are manually tuned for 512x512 images.
    * **Future Fix**: Implement adaptive or local thresholding methods (e.g., Sauvola or Otsu) for broader applicability.
2.  **Fixed Clustering Radius**:
    * **Current Fix**: A static 6px merge radius is used.
    * **Future Fix**: Use connected-component labeling to find centroids or a dynamic radius based on local line density.
3.  **Axis Handling**:
    * **Current Fix**: Axes are implicitly ignored by skipping a 3px border and using the `7x7 <= 30` pixel rule.
    * **Future Fix**: Explicitly detect and remove axes using techniques like the Hough transform or longest-line removal.

### Alternative Solution: Graph Skeletonization 
* A more robust, general-purpose method involves using morphological thinning to reduce the graph lines to a 1-pixel wide skeleton.
* This skeleton can be treated as a mathematical graph, where intersection points are simply nodes with a degree of 3 or more.
* This approach scales well to more complex diagrams and network maps.
