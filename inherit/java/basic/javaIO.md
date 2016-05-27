[TOC]
# java io 基础

## Java 位运算

在Java流基础中，存在java位运算的一些机制，因此好好掌握一下

- **正数负数**
正数和负数都是取反+1 ，方便加法运算
例如 +6 二进制（不考虑32位，考虑8位差不多）
+6:    0000 0110 	
取反: 1111 1001
-6:     1111 1010	
这样 +6 和 -6 进行加法（全部消去），1 0000 0000 超出8位是0

- **有符号移位**
	在一定范围内左移和右移可以看成是对2的乘法和除法运算（看成a[n-1]...a[0]，其中a[i]是0或者1，很容易证明）
	
	*负数左移用0补位*
	```java
		11111111 11111111 11111111 11111010  //-6 左移1位
		11111111 11111111 11111111 11110100  //12
	```
	*负数右移用1补位*
	```java
		11111111 11111111 11111111 11111010  //-6 右移1位
		11111111 11111111 11111111 11111101  //-3
	```
	*正数数右移用0补位*
	```java
		110  //6 右移1位
		11  //3
	```
	*正整数左移用0补位*
	```java
		110  //6 左移1位
	   1100  //12
	```
- **无符号移位** 好像只有右移

	*负数右移用0补位*
	```java
		11111111 11111111 11111111 11111010  //-6 右移1位
		01111111 11111111 11111111 11111101  //2147483645
	```
	*正数数右移用0补位*
	```java
		110  //6 右移1位
		11  //3
	```

- **&运算**
	第一个操作数的的第n位于第二个操作数的第n位如果都是1，那么结果的第n为也为1，否则为0
	```java
		//例如用0xff来转化byte为无符号整数
		//-1 -> 255
		public static void formatByte(byte b){
			return b & 0xff;
		}
	```

## copy : input -> output (io基础)

**InputStream 和 Reader**
java.io下面有两个抽象类：InputStream和Reader
**InputStream**:是表示字节输入流的所有类的超类
**Reader**:是用于读取字符流的抽象类
1. InputStream提供的是字节流的读取，而非文本读取，这是和Reader类的根本区别。
2. 即用Reader读取出来的是char数组或者String ，使用InputStream读取出来的是byte数组。
3. 也因为是文本读取，所以存在编码问题

**理解通用方法**
`available()`: 并不是指定文件还有多少数据没有读，不然，如果是100g的大文件，返回得数就溢出了。不过，对于普通文件的话可以看做是剩余的未读数据。
返回下一次对此输入流调用的方法可以不受阻塞地从此输入流读取（或跳过）的估计剩余字节数。

`mark()` : `this.mark = this.pos;`
`reset()`:`this.pos = this.mark;`



**理解read方法**

-  1.**抽象类 `InputStream`**的`read`方法
```java
	/**
	* 读取len长度的字节到byte[]中，读的长度不一定是len(如果已经到文件末尾)	
	* @param off : 起始位置
	* @param b : 缓存数组
	* @param len : 读取的长度
	**/
	public int read(byte b[], int off, int len) throws IOException {
		//1. 参数的验证
        if (b == null) {
            throw new NullPointerException();
        } else if (off < 0 || len < 0 || len > b.length - off) {
            throw new IndexOutOfBoundsException();
        } else if (len == 0) {
            return 0;
        }
		//2 .读取，如果文件已经文件末尾，返回-1
        int c = read();
        if (c == -1) {
            return -1;
        }
        //3 .开始读取并且填充道缓存byte数组中
        b[off] = (byte)c;
		
        int i = 1;
        try {
            for (; i < len ; i++) {
                c = read();
                if (c == -1) {
                    break;
                }
                b[off + i] = (byte)c;
            }
        } catch (IOException ee) {
        }
        return i;
    }
	
	/**
	* 抽象方法，具体实现由子类实现，比如FileInputStream
	* 返回下一个byte
	* the next byte of data, or <code>-1</code> if the end of the
    *             stream is reached.
	**/
	public abstract int read() throws IOException;
```


- 2.`copy large`方法 -基于流Stream
```java

private static byte[] SKIP_BYTE_BUFFER;
private static final int DEFAULT_BUFFER_SIZE = 1024 * 4;
private static final int SKIP_BUFFER_SIZE = 2048;

public static long copyLarge(InputStream input, OutputStream output, byte[] buffer)
            throws IOException {
        long count = 0;
        int n = 0;
        while (true) {
            // input 读取的字节数
            n = input.read(buffer);
            if (n == -1)
                break;
            output.write(buffer, 0, n);
            count = count + n;
        }
        return count;
    }
/**
     * @param input       输入流
     * @param output      输出流
     * @param inputOffset 起始位置
     * @param length      需要读取的长度
     * @param buffer      缓存数组
     * @return
     * @throws IOException
     */
    public static long copyLarge(InputStream input,
                                 OutputStream output,
                                 final long inputOffset,
                                 final long length,
                                 byte[] buffer) throws IOException {
        //1 .如果直接需要跳过字节
        if (inputOffset > 0) {
            skipFully(input, inputOffset);
        }
        if (length == 0) {
            return 0;
        }
        final int bufferLength = buffer.length;
        int bytesToRead = bufferLength;
        //2.1 如果需要读取的长度小于缓存数组的长度，读取长度是2者中最小值
        if (length > 0 && length < bufferLength) {
            bytesToRead = (int) length;
        }
        int read;
        long totalRead = 0;
        //3. 循环copy
        while (true) {
            //3.1 如果不用读取了，中断，即已经读取了这么多长度
            if (bytesToRead <= 0) {
                break;
            }
            //3.2 读取道末尾页中断
            read = input.read(buffer, 0, bytesToRead);
            if (read == EOF) {
                break;
            }
            // 3.3 否则的化，进行复制
            output.write(buffer, 0, read);
            totalRead += read;
            // 3.4 读完之后，判断下次读取多少，不想读太多
            if (length > 0) {
                bytesToRead = (int) Math.min(length - totalRead, bufferLength);
            }
        }

        return totalRead;
    }


    public static void skipFully(InputStream input, long toSkip) throws IOException {
        //1. 如果要skip的数目<0 ， 异常
        if (toSkip < 0) {
            throw new IllegalArgumentException("Bytes to skip must not be negative: " + toSkip);
        }
        long skipped = skip(input, toSkip);
        //2.如果希望跳过的字节数和实际跳过字节数不相同，例如要跳过73个字节，但是文件长度只有72个，抛出异常
        if (skipped != toSkip) {
            throw new EOFException("Bytes to skip: " + toSkip + " actual: " + skipped);
        }
    }

    public static long skip(InputStream input, long toSkip) throws IOException {
        if (toSkip < 0) {
            throw new IllegalArgumentException("Skip count must be non-negative, actual: " + toSkip);
        }
        // 如果skip的 buffer是空，创建一个数组存放
        if (SKIP_BYTE_BUFFER == null) {
            SKIP_BYTE_BUFFER = new byte[SKIP_BUFFER_SIZE];
        }
        long remain = toSkip;
        while (true) {
            if (remain <= 0) {
                break;
            }
            //读取的长度是多少呢?
            int len2Read = (int) Math.min(remain, SKIP_BUFFER_SIZE);
            long n = input.read(SKIP_BYTE_BUFFER, 0, len2Read);
            if (n < 0) {
                //已经是文件末尾
                break;
            }
            remain = remain - n;
        }
        return toSkip - remain;
    }
```


## 通过ByteArrayInputStream和ByteArrayOutStream理解Stream

ArrayInputStream非常简单，以流的方式来看待byte[]

```java
	package com.carl.demo.io;

import java.io.Closeable;
import java.io.IOException;
import java.io.InputStream;
import java.util.Arrays;

/**
 * Created by carl on 16-3-1.
 */
public class MyByteArrayInputStream extends InputStream implements Closeable {
    /**
     * 一个由流创建者提供的byte数组.
     * 整个byte数组的元素只能从流中读取
     */
    protected byte[] buf;

    protected int pos;

    protected int count;

    protected int mark;


    public MyByteArrayInputStream(byte[] buf) {
        this.buf = buf;
        this.pos = 0;
        this.count = buf.length;
    }


    public MyByteArrayInputStream(byte[] buf, int offset, int length) {
        this.buf = buf;
        this.pos = offset;
        this.count = Math.min(offset + length, buf.length);
        this.mark = offset;
    }


    @Override
    public synchronized int read() throws IOException {
        if (pos < count) {
            byte b = buf[pos++];
            return b & 0xff;
        }
        //pos>=count 表示超出范围
        return -1;
    }

    /**
     * 从buf中写len个字段
     *
     * @param b
     * @param off
     * @param len
     * @return
     */
    public synchronized int read(byte b[], int off, int len) {
        if (b == null) {
            throw new NullPointerException();
        } else if (off < 0 || len < 0 || len > b.length - off) {
            throw new IndexOutOfBoundsException();
        }
        // 如果pos位置超过了count，超出了buf的大小，返回-1？
        if (pos >= count) {
            return -1;
        }
        // 还剩下多少可用的
        int avail = count - pos;
        if (len > avail) {
            len = avail;
        }

        if (len <= 0) {
            return 0;
        }
        System.arraycopy(buf, pos, b, off, len);
        pos += len;
        return len;
    }

    /**
     * 返回这次可以读到的数据
     *
     * @return
     */
    public synchronized int available() {
        return count - pos;
    }

    public boolean markSupported() {
        return true;
    }

    /**
     * 标记到当前位置
     * @param readAheadLimit
     */
    public void mark(int readAheadLimit) {
        mark = pos;
    }

    /**
     * 充值位置，pos指向标记位置
     */
    public synchronized void reset() {
        pos = mark;
    }

    public void close() throws IOException {
    }

    public static void main(String[] args) {
        byte[] buf = new byte[127];
        for (int i = 0; i < 127; i++) {
            buf[i] = (byte) i;
        }

        int offset = 0;
        int length = 101;
        MyByteArrayInputStream in = new MyByteArrayInputStream(buf, offset, length);
        byte[] b = new byte[20];
        int i = 1;
        while (true) {
            int n = in.read(b, 0, b.length);
            System.out.println(b);
            System.out.println(i + ":" + Arrays.toString(b));
            System.out.println("读了:" + n + "个");
            i++;
            if (n < 0) {
                break;
            }
        }
    }

}

```


ArrayOutputStream就相对复杂,下面介绍是Apache出品的ByteArrayOutputStream
其中使用了`List<byte[]> buffers`

```java
package org.apache.commons.io.output;
 
import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.SequenceInputStream;
import java.io.UnsupportedEncodingException;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import org.apache.commons.io.input.ClosedInputStream;
public class ByteArrayOutputStream extends OutputStream {

    /** A singleton empty byte array. */
    private static final byte[] EMPTY_BYTE_ARRAY = new byte[0];

    /** The list of buffers, which grows and never reduces. */
    private final List<byte[]> buffers = new ArrayList<byte[]>();
    /** The index of the current buffer. */
    private int currentBufferIndex;
    /** The total count of bytes in all the filled buffers. */
    private int filledBufferSum;
    /** The current buffer. */
    private byte[] currentBuffer;
    /** The total count of bytes written. */
    private int count;

    /**
     * Creates a new byte array output stream. The buffer capacity is 
     * initially 1024 bytes, though its size increases if necessary. 
     */
    public ByteArrayOutputStream() {
        this(1024);
    }

    /**
     * Creates a new byte array output stream, with a buffer capacity of 
     * the specified size, in bytes. 
     *
     * @param size  the initial size
     * @throws IllegalArgumentException if size is negative
     */
    public ByteArrayOutputStream(int size) {
        if (size < 0) {
            throw new IllegalArgumentException(
                "Negative initial size: " + size);
        }
        synchronized (this) {
            needNewBuffer(size);
        }
    }

    /**
     * Makes a new buffer available either by allocating
     * a new one or re-cycling an existing one.
     *
     * @param newcount  the size of the buffer if one is created
     */
    private void needNewBuffer(int newcount) {
        if (currentBufferIndex < buffers.size() - 1) {
            //Recycling old buffer
            filledBufferSum += currentBuffer.length;
            
            currentBufferIndex++;
            currentBuffer = buffers.get(currentBufferIndex);
        } else {
            //Creating new buffer
            int newBufferSize;
            if (currentBuffer == null) {
                newBufferSize = newcount;
                filledBufferSum = 0;
            } else {
                newBufferSize = Math.max(
                    currentBuffer.length << 1, 
                    newcount - filledBufferSum);
                filledBufferSum += currentBuffer.length;
            }
            
            currentBufferIndex++;
            currentBuffer = new byte[newBufferSize];
            buffers.add(currentBuffer);
        }
    }

    /**
     * Write the bytes to byte array.
     * @param b the bytes to write
     * @param off The start offset
     * @param len The number of bytes to write
     */
    @Override
    public void write(byte[] b, int off, int len) {
        if ((off < 0) 
                || (off > b.length) 
                || (len < 0) 
                || ((off + len) > b.length) 
                || ((off + len) < 0)) {
            throw new IndexOutOfBoundsException();
        } else if (len == 0) {
            return;
        }
        synchronized (this) {
            int newcount = count + len;
            int remaining = len;
            int inBufferPos = count - filledBufferSum;
            while (remaining > 0) {
                int part = Math.min(remaining, currentBuffer.length - inBufferPos);
                System.arraycopy(b, off + len - remaining, currentBuffer, inBufferPos, part);
                remaining -= part;
                if (remaining > 0) {
                    needNewBuffer(newcount);
                    inBufferPos = 0;
                }
            }
            count = newcount;
        }
    }

    /**
     * Write a byte to byte array.
     * @param b the byte to write
     */
    @Override
    public synchronized void write(int b) {
        int inBufferPos = count - filledBufferSum;
        if (inBufferPos == currentBuffer.length) {
            needNewBuffer(count + 1);
            inBufferPos = 0;
        }
        currentBuffer[inBufferPos] = (byte) b;
        count++;
    }

    /**
     * Writes the entire contents of the specified input stream to this
     * byte stream. Bytes from the input stream are read directly into the
     * internal buffers of this streams.
     *
     * @param in the input stream to read from
     * @return total number of bytes read from the input stream
     *         (and written to this stream)
     * @throws IOException if an I/O error occurs while reading the input stream
     * @since 1.4
     */
    public synchronized int write(InputStream in) throws IOException {
        int readCount = 0;
        int inBufferPos = count - filledBufferSum;
        int n = in.read(currentBuffer, inBufferPos, currentBuffer.length - inBufferPos);
        while (n != -1) {
            readCount += n;
            inBufferPos += n;
            count += n;
            if (inBufferPos == currentBuffer.length) {
                needNewBuffer(currentBuffer.length);
                inBufferPos = 0;
            }
            n = in.read(currentBuffer, inBufferPos, currentBuffer.length - inBufferPos);
        }
        return readCount;
    }

    /**
     * Return the current size of the byte array.
     * @return the current size of the byte array
     */
    public synchronized int size() {
        return count;
    }

    /**
     * Closing a <tt>ByteArrayOutputStream</tt> has no effect. The methods in
     * this class can be called after the stream has been closed without
     * generating an <tt>IOException</tt>.
     *
     * @throws IOException never (this method should not declare this exception
     * but it has to now due to backwards compatability)
     */
    @Override
    public void close() throws IOException {
        //nop
    }

    /**
     * @see java.io.ByteArrayOutputStream#reset()
     */
    public synchronized void reset() {
        count = 0;
        filledBufferSum = 0;
        currentBufferIndex = 0;
        currentBuffer = buffers.get(currentBufferIndex);
    }

    /**
     * Writes the entire contents of this byte stream to the
     * specified output stream.
     *
     * @param out  the output stream to write to
     * @throws IOException if an I/O error occurs, such as if the stream is closed
     * @see java.io.ByteArrayOutputStream#writeTo(OutputStream)
     */
    public synchronized void writeTo(OutputStream out) throws IOException {
        int remaining = count;
        for (byte[] buf : buffers) {
            int c = Math.min(buf.length, remaining);
            out.write(buf, 0, c);
            remaining -= c;
            if (remaining == 0) {
                break;
            }
        }
    }

    /**
     * Fetches entire contents of an <code>InputStream</code> and represent
     * same data as result InputStream.
     * <p>
     * This method is useful where,
     * <ul>
     * <li>Source InputStream is slow.</li>
     * <li>It has network resources associated, so we cannot keep it open for
     * long time.</li>
     * <li>It has network timeout associated.</li>
     * </ul>
     * It can be used in favor of {@link #toByteArray()}, since it
     * avoids unnecessary allocation and copy of byte[].<br>
     * This method buffers the input internally, so there is no need to use a
     * <code>BufferedInputStream</code>.
     * 
     * @param input Stream to be fully buffered.
     * @return A fully buffered stream.
     * @throws IOException if an I/O error occurs
     * @since 2.0
     */
    public static InputStream toBufferedInputStream(InputStream input)
            throws IOException {
        ByteArrayOutputStream output = new ByteArrayOutputStream();
        output.write(input);
        return output.toBufferedInputStream();
    }

    /**
     * Gets the current contents of this byte stream as a Input Stream. The
     * returned stream is backed by buffers of <code>this</code> stream,
     * avoiding memory allocation and copy, thus saving space and time.<br>
     * 
     * @return the current contents of this output stream.
     * @see java.io.ByteArrayOutputStream#toByteArray()
     * @see #reset()
     * @since 2.0
     */
    private InputStream toBufferedInputStream() {
        int remaining = count;
        if (remaining == 0) {
            return new ClosedInputStream();
        }
        List<ByteArrayInputStream> list = new ArrayList<ByteArrayInputStream>(buffers.size());
        for (byte[] buf : buffers) {
            int c = Math.min(buf.length, remaining);
            list.add(new ByteArrayInputStream(buf, 0, c));
            remaining -= c;
            if (remaining == 0) {
                break;
            }
        }
        return new SequenceInputStream(Collections.enumeration(list));
    }

    /**
     * Gets the curent contents of this byte stream as a byte array.
     * The result is independent of this stream.
     *
     * @return the current contents of this output stream, as a byte array
     * @see java.io.ByteArrayOutputStream#toByteArray()
     */
    public synchronized byte[] toByteArray() {
        int remaining = count;
        if (remaining == 0) {
            return EMPTY_BYTE_ARRAY; 
        }
        byte newbuf[] = new byte[remaining];
        int pos = 0;
        for (byte[] buf : buffers) {
            int c = Math.min(buf.length, remaining);
            System.arraycopy(buf, 0, newbuf, pos, c);
            pos += c;
            remaining -= c;
            if (remaining == 0) {
                break;
            }
        }
        return newbuf;
    }

    /**
     * Gets the curent contents of this byte stream as a string.
     * @return the contents of the byte array as a String
     * @see java.io.ByteArrayOutputStream#toString()
     */
    @Override
    public String toString() {
        return new String(toByteArray());
    }

    /**
     * Gets the curent contents of this byte stream as a string
     * using the specified encoding.
     *
     * @param enc  the name of the character encoding
     * @return the string converted from the byte array
     * @throws UnsupportedEncodingException if the encoding is not supported
     * @see java.io.ByteArrayOutputStream#toString(String)
     */
    public String toString(String enc) throws UnsupportedEncodingException {
        return new String(toByteArray(), enc);
    }

}

```
