+++
title = "UVA 10920 Spiral Tap"
comments = true
draft = false
showcomments = true
showpagemeta = true
categories = ["algorithms"]
tags = []
date = "2014-07-04T10:17:55+08:00"
slug = ""

+++

A week gone by without tackling an algorithm problem makes Jack a dull software engineer.

Lately, I’ve been working on an inhouse app which requires intermediate Javascript and CSS skills. Being the greenhorn I am at these technologies, I struggled and the process became a timesink. Hence I didn’t manage to set aside time for my regular algorithm coding session.

Prior to this busy week, I encountered the [eponymous algorithm problem](http://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=1861) on UVa. Yes, I’m back on Uva OJ again. I found out that the slow response times of UVa OJ previously was due to problems with my browser cache. I am able to access the site using Incognito Mode on Chrome, since the browswer loads the site from a clean slate. There is truly no good subsitute for the problem classifier on UVa. CodeChef has its upsides – good community and support for more languages, but when it comes to finding questions by topic, it has nil support for that.

Spiral Tap is an interesting ad-hoc problem. The premise is quite straight-forward: the numbering sequence in the grid follows a spiral pattern that originates from the center square, and given a square number, you are required to find out the coordinates of that square. The [problem statement](http://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=1861) has a diagram to illustrate the spiral-patterned grid.

It isn’t hard to visualize the solution through the naive approach. What you can do is to construct a 2D array first. Starting from the center square, you number each square from 1 to N using the spiral pattern. Given a square number from the input, you then find the coordinates of that square (note that the coordinates specified in the question is a bit special) and output it as the answer.

So the naive me, prior to this busy week, coded the naive solution and found out that my program sputtered and stopped upon using test cases of extremely large grids. Grids of size 10000x10000. Mind you. Upon realising that the naive solution is useless, I was quite disheartened at the prospect of finding a mathematical formula to solve the problem and decided to shelve the problem for the next week.

Luckily I chanced upon a hint given by a kind dude on the UVa discussion forum, it was just a hint and no more (not to the extent of spoonfeeding). I couldn’t say it better, so you can check out the hint [here](http://acm.uva.es/board/viewtopic.php?p=50676&sid=03fc5df83f88ad1a3077f6abddf28fcc#p50676).

I managed to squeeze out some time this week to code up the solution. It wasn’t easy, because while his hint describes the solution in layman terms, I couldn’t nail down the programmatic version. The hard part is, how do you find out which of the four lines the square is in? And more importantly, how do you find the coordinates based on the square’s position along the line?

So after quite some time thinking and doodling on paper, I managed to find a pattern. Supposed we want to find the coordinates of square number 15 in a 5x5 grid. Based on his hint, 15 falls in between 9 and 25, four lines connects square 9 and square 25. Which line contains square 15 and exactly where in the line is 15?

Fret not. First, we enumerate all the square numbers and their coordinates between square 9 and square 25:

	9 (5,5)

	10 (6,5)
	11 (6,4)
	12 (6,3)
	13 (6,2)

	14 (5,2)
	15 (4,2)
	16 (3,2)
	17 (2,2)

	18 (2,3)
	19 (2,4)
	20 (2,5)
	21 (2,6)

	22 (3,6)
	23 (4,6)
	24 (5,6)
	25 (6,6)

Here’s the thing. Clearly there are four lines, the squares in each line has a fixed coordinate on either the X or Y side. For the other side of the coordinates that is not fixed, it follows a incremental/decremental pattern as the square number increases.

So the algorithm is becoming much clearer now. We need to find the square numbers that mark the start of each line segment, in this case, they are squares 10, 14, 18 and 22. It’s trivial to calculate that 15 falls between 14 and 18. Based on the pattern that the second line segment’s x-coordinate follows a decremental pattern and that it has a fixed y-coordinate of 2, we start at (5, 2) at square 14 and logically we conclude that (4, 2) is the coordinate of square 15.

Having known the pattern, the solution is pretty much easier to visualize and program now.

Since this is an ad-hoc problem, the solution can’t really be generalized to solve other types of 2D array problems. But what I learnt from this coding session is that finding patterns can be a viable way to generate algorithms to solve a problem. Sometimes you just have to do it the hard way, to enumerate the steps so as to glean the pattern from the details.

My solution (AC):

```java
import java.io.*;

/**
 * @author Alex Jiao
 * UVa 10920 Spiral Tap
 *
 */
class Main {
  public static void main(String args[]) {
      DataReader in = new DataReader(System.in);
      PrintWriter out = new PrintWriter(System.out);
      int size, pos;
      do {
          size = in.nextInt();
          pos = in.nextInt();
          if (pos==0 && size==0) {
              break;
          }
          int baseCoord = (int) Math.ceil(size/2.0);
          int baseSqrt = (int) Math.sqrt(pos);
          if (baseSqrt%2==0) {
              baseSqrt -= 1;
          }
          int nextSqrt = baseSqrt + 2;
          int nextSq = (int)Math.pow(nextSqrt,2);
          int baseSq = (int)Math.pow(baseSqrt,2);
          baseCoord += (baseSqrt-1)/2;
          int maxCoord = baseCoord + 1;
          int minCoord = baseCoord - baseSqrt;
          int x, y, secSegMarker,thirdSegMarker, fourthSegMarker;
          int segLength = (nextSq-baseSq)/4;
          secSegMarker = baseSq + segLength + 1;
          thirdSegMarker = baseSq + 2 * segLength + 1;
          fourthSegMarker = baseSq + 3 * segLength + 1;
          
          if (pos > baseSq && pos < secSegMarker) {
              x = maxCoord;
              y = (maxCoord-1) - (pos - (baseSq + 1));
          } else if(pos>= secSegMarker && pos < thirdSegMarker) {
              y = minCoord;
              x = (maxCoord-1) - (pos - secSegMarker);
          } else if(pos>=thirdSegMarker && pos < fourthSegMarker) {
              x = minCoord;
              y = minCoord + 1 + (pos - thirdSegMarker);
          } else if(pos>=fourthSegMarker && pos <= nextSq) {
              y = maxCoord;
              x = minCoord + 1 + (pos - fourthSegMarker);
          } else {
              x = baseCoord;
              y = baseCoord;
          }    
          out.printf("Line = %d, column = %d.\n", x, y);
      } while(true);
      out.close();
  }
  
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