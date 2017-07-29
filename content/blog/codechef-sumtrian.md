+++
title = "CodeChef - SUMTRIAN"
showpagemeta = true
categories = ["algorithms", "dynamic programming"]
date = "2014-06-21T10:07:42+08:00"
draft = false
showcomments = true
tags = []
slug = ""
comments = true

+++

I’ve been doing some algorithm problems lately. It helps to keep you sharp between the ears and makes you a better engineer overall. The only downside is the time-consuming nature of this pursuit, as I am not too proficient in churning out optimized solutions.

I started by doing problems using UVa Online Judge, the good thing about this site is the immense archive of problems and the [problem classifier](http://uhunt.onlinejudge.org/) maintained by Felix Halim. But there’s really one bad thing about UVa, its site is so slow and unstable.

Hence I switched over to CodeChef. It has a better community and support for more languages (you can even code in Scala!), most importantly, the judging process is really fast. No more waiting over 5 minutes to get your verdict.

Today I decided to tackle my first problem on CodeChef. I chose a DP-related problem in the Easy category because I’ve been hearing about dynamic programming for a long time and I don’t really know what it is.

Turns out it is just recursion + memoization (caching of results).

I know my understanding of DP is overly-simplistic, but it is a good enough mental model to solve additional DP-related problems.

Problem statement: http://www.codechef.com/problems/SUMTRIAN

For this problem, it is about obtaining the max sum by summing a value on each of the triangle’s levels. To sum the values, you have to traverse from the apex to the base; during the traversal, you can only go down and down-right.

I submitted a recursive-only solution and got back a TLE (Time Limit Exceeded) result, but when I used memoization to cache the results of the recursive calls, my solution was accepted.

Turns out the number of recursive calls to find the max sums at each level of the triangle grows at an exponential rate when the input is large. That’s why my recursive-only solution exceeded the time limit.

But when you cache the results of the previous recursive calls in a 2D array, you can access the previous results at O(1) time without calling the same recursive function again. This technique is called memoization and it makes your algorithm really fast.

That’s why today’s coding session is really fruitful, I learnt what it means by dynamic programming – approaching a problem by implementing it as a normal recursive solution, and then to add the caching part.

Recursive + caching solution (AC):

```java
import java.io.IOException;
import java.io.InputStream;
import java.io.PrintWriter;

/**
 * @author Alex Jiao
 *
 */
public class SUMTRIAN {
  public static void main(String args[]) throws NumberFormatException, IOException {
      DataReader in = new DataReader(System.in);
      PrintWriter out = new PrintWriter(System.out);
      int numTests = in.nextInt();
      while (numTests > 0) {
          int height = in.nextInt();
          int[][] triangle = new int[height][];
          for (int i=0; i<height; i++) {
              triangle[i] = new int[i + 1];
              for (int j=0; j<=i; j++) {
                  triangle[i][j] = in.nextInt();
              }
          }
          // cache used for memoization
          int[][] maxSums = new int[100][100];
          int ans = findMaxSum(triangle, 0, 0, maxSums);
          out.println(ans);
          numTests--;
      }
      out.close();
  }
  
    private static int findMaxSum(int[][] triangle, int height, int index, int[][]maxSums) {
      if (height == triangle.length-1) {
          return triangle[height][index];
      }
      else {
          int max1, max2;
          // caching of max sums
          if (maxSums[height+1][index] == 0) {
              maxSums[height+1][index] = findMaxSum(triangle, height+1, index, maxSums);
          }
          if (maxSums[height+1][index+1] == 0) {
              maxSums[height+1][index+1] = findMaxSum(triangle, height+1, index+1, maxSums);
          }
          
          max1 = maxSums[height+1][index];
          max2 = maxSums[height+1][index+1];
          
          return triangle[height][index] +
                  Math.max(max1, max2);
      }
  }
    // custom-made input reader which is really fast (not coded by me)
  static class DataReader {
        private InputStream in = null;
        private int pos, count;
        private byte[] buf = new byte[64 * 1024];

        public DataReader(InputStream in) {
            this.in = in;
            pos = 0;
            count = 0;
        }

        public int nextInt() {
            int c, sign = 1;
            while (Character.isWhitespace(c = read())) ;
            if (c == '-') {
                sign = -1;
                c = read();
            }
            int n = c - '0';
            while ((c = read() - '0') >= 0)
                n = n * 10 + c;
            return n * sign;
        }

        public String next() {
            StringBuilder builder = new StringBuilder();
            int c;
            while (!Character.isSpaceChar(c = read()))
                builder.append(c);
            return builder.toString();
        }

        public String nextLine() {
            StringBuilder builder = new StringBuilder();
            int c;
            while ((c = read()) != '\n')
                builder.append(c);
            return builder.toString();
        }

        public int read() {
            if (pos == count)
                fillBuffer();
            return buf[pos++];
        }

        private void fillBuffer() {
            try {
                count = in.read(buf, pos = 0, buf.length);
                if (count == -1)
                    buf[0] = -1;
            } catch (Exception ignore) {
            }
        }
    }
}
```