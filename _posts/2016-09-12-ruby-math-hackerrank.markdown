---
layout: post
title: Ruby Math Hackerrank
---

# Install

Install vim plugin [seeing-is-believing]({{ site.baseurl }}
{% post_url 2015-12-01-vim-tips %}#seeing-is-believing) and write Gemfile

~~~
# Gemfile
source 'https://rubygems.org'

ruby '2.2.3'

# for instant output of commands
gem 'seeing_is_believing'

# irbtools includes interactive_editor (vim inside irb)
# just create ~/.irbrc with
# require 'rubygems'
# require 'irbtools'
gem 'irbtools'

# debugging
gem 'byebug'

# simple test assertions
gem 'minitest'
~~~

# Links

Math and algorith tasks

* https://www.hackerrank.com
  * nice editor, solution have to be memory effiction (it raises Timeout or Run
  Time Error)
* https://projecteuler.net/
* http://www.beatmycode.com/
* http://codility.com/programmers/

Codility Solutions

* https://github.com/GNakayama/codility/tree/master/src/python and blog
  http://blog.nkbits.com/
* http://codesays.com/ http://codesays.com/solutions-to-training-by-codility/ 
* http://www.martinkysel.com/codility-solutions/

# Just enough ruby

* read from STDIN or from a file if you run with `ruby script.rb my_data`

  ~~~
  n = gets.to_i
  array = gets.strip.split(' ').map(&:to_i)
  ~~~

* write to a file

  ~~~
  File.open "my_file_name", 'w' do |file|
    file.puts "first line (no need for new line char \ n)"
    file << "second line \n"
    file.write "third line \n"
  end
  ~~~

* division is integer division, so that `1/2 == 0` in ruby. You need to convert
  to float `x = x.to_f`
* iterate range `(1..10).each` or `1.upto(10).each`
* exponent is `2**2 == 4`

# Example code

We can use [pp](https://ruby-doc.org/stdlib-2.1.0/libdoc/pp/rdoc/PP.html)
standard library to print arrays just `require 'pp'`. But if you need to print
matrix, than you can use ppp `require './ppp'`

~~~
# ppp.rb
def ppp(x)
  if x.class == Array
    puts_array(x)
  else
    puts x
  end
end

def puts_array(x)
  need_new_line = false
  x.each do |e|
    if e.class == Array
      puts e.join(', ')
    else
      print e.to_s + ', '
      need_new_line = true
    end
  end
  puts if need_new_line
end
~~~

Here is example for one solution that is submitted on site (it can not
`require byebug`) so it use constant PRODUCTION. Test are in the same file.
If the task is to read from file and to submit generated file, than use second
tempalte.

~~~
# 0_short_palindrome.rb
# https://www.hackerrank.com/challenges/short-palindrome
# string s of n lowecase letters s_i 0<=i<n
# (a, b, c, d)
# s_a = s_b chars at a and d are the same
# s_b = s_c
# constrains:
# 0<=a<b<c<d<n
# input: string s
# 1<= |s| < 10^6
# output: number of (a, b, c, d) tuples, use moduo 10^9 + 7
PRODUCTION = false
if PRODUCTION
  def ppp(_arg = nil); end

  def pp(_arg = nil); end
else
  require 'pp'
  require './ppp'
  require 'byebug'
end

# rubocop:disable Metrics/MethodLength
# rubocop:disable Metrics/AbcSize
def solution(a)
  r = a.length
  ppp "r=#{r}"
  r
end

if PRODUCTION
  n = gets.to_i
  a = []
  n.times do
    # strip is needed since on hackerrank, new line is at the end of gets
    r = gets.strip.split('').map { |c| c == '*' ? 0 : 1 }
    a << r
  end
  puts solution(a)
end # if PRODUCTION

require 'minitest/autorun'
class Test < Minitest::Test
  def test_sample
    s = 'kkkkkkz'
    assert_equal 12, solution(prepare(s))
  end

  def test_long
    s = %(***.*
**..*
***.*
*****
*****)
    assert_equal 12, solution(prepare(s))
  end

  def test_float
    asert_in_delta 0.5, solution(1,2) # default delta=0.001
  end

  def test_that_is_skipped
    skip "later"
  end

  private

  def prepare(s)
    s
  end
end
~~~

This template has separate test and actual code

~~~
# my_script.rb
# description of a problem
require 'pp'
require './ppp'
require 'byebug'

def solution(aa:, l:, h:)
end

# when we run: ruby my_script.rb data/example.in
# output is saved in data/example.out
if ARGV.length == 1
  file_name = ARGV.last.split('.').first
  r, _c, l, h = gets.split(' ').map(&:to_i)
  aa = []
  r.times do
    # strip is needed since on hackerrank, new line is at the end of gets
    aa << gets.strip.split('').map { |c| c == 'M' ? 1 : 0 }
  end
  results = solution(aa: aa, l: l, h: h)
  File.open "#{file_name}.out", 'w' do |file|
    # puts results.length
    file.puts results.length
    results.each do |row|
      # puts row.join(' ')
      file.puts row.join(' ')
    end
  end
end
~~~

~~~
# test_my_script.rb
require 'minitest/autorun'
require './pizza'
class Test < Minitest::Test
  def test_solution
    a =
    [
      1,
    ]
    res = 1
    assert_equal res, solution(a)
  end
end
~~~

If you want to run only one test, than run `ruby 0_short_palindrome.rb --name
test_sample`

If you want to print all local variables, you can use

~~~
local_variables.map { |vn| { vn => eval(vn.to_s) } }
~~~

Run with colors `ruby my_class_test.rb -p`, or `ruby -r
minitest/pride my_class_test.rb` or put `require 'pride'` in test file.

For minitests you can use `describe` and `it` blocks.

TIPS:

* if you need to perform big number of tests, and you can not calculate solution
  but you need to iterate something, than use table of solutions. First call
  `create_table` of results and query that `@table[n]` (if you can, fill just
  what it needs)
* two dimensional 2d array `@table = Array.new(n) { Array.new(n) }`
* if stack level too deep then try to use `@table` class variables instead of
  passing them as arguments (not much help since arguments are passed by
  reference, but there are less number of arguments
  [link](http://stackoverflow.com/questions/28703587/systemstackerror-when-pushing-more-than-130798-objects-into-an-array))

# Complexity

Ruby `Array#find` method has O(n) complexity. If array is big, you should use
[bsearch](http://ruby-doc.org/core-2.1.5/Array.html#method-i-bsearch) O(log(n)).
Array needs to be sorted. Binary search works in two modes:
* find minimum: block must return true or false and there must be an index `i`
  so that: block returns false elements whose index is less than `i`, block
  returns true for any element whose index is greater than or equal to `i`.
  This method returns `i`-th element.
* find any: block returns number and there are must be two indices `i` and `j`
  (`0<=i<=j<=ary.size`) so that: block returns positive number for `0<=k<i`,
  returns zero for `i<=k<j` and negative for `j<=k<ary.size`.
  This method returns any element whose index is within `i..j`.

~~~
a = [1, 4, 7, 10]
# find minimum that satisfy block
a.bsearch { |x| x >= 6 } #=> 7

# find any
ary = [0, 4, 7, 10, 12]
# try to find v such that 4 <= v < 8
ary.bsearch {|x| 1 - x / 4 } #=> 4 or 7
# try to find v such that 8 <= v < 10
ary.bsearch {|x| 4 - x / 2 } #=> nil
~~~

Array `a.include?` is O(n) but hash `a.key?` (or `a.include?`) is O(1). So
instead of big time, you can use big space by using Hashtables.

# Counting of elements

[tutorial](https://codility.com/media/train/2-CountingElements.pdf)
if we need to check if some elements exists, we can create counting array in
O(N) time and query in constant time

~~~
a = [0, 3, 1, 3]
counting = [0]*(a.max+1)
a.each { |e| counting[e]+=1 }
counting # =>  [1, 1, 0, 2]
counting[2] == 0 # true in constant time
~~~

# Prefix sums

[tutorial](https://codility.com/media/train/3-PrefixSums.pdf) In O(N) time we
can calculate p_1, ...p_n, where p_k= a_0+a_1+...+a_(k-1). Note that p_k does
not use a_k, and p_0 == 0 Than we can calculate sum of any slice (x,y) in
constant time instead of O(y-x) (that is usually needed for query several
results).  Slice could be defined on 1..n (index starts from 1 and include
bounds), slice(x,y) = prefix_sum[y] - prefix_sum[x-1] for example
~~~
slice(a_0,a_0) = slice(1,1) = prefix_sum[1]-prefix_sum[0]
slice(a_1,a_3) = slice(2,4) = prefix_sum[4]-prefix_sum[1]
~~~

~~~
a = [1, 4, 0, 4, 2]
prefix_sum = [0] * (a.length+1)
a.each_with_index do |e,i|
  prefix_sum[i+1] = prefix_sum[i] + e
end
# slice a_1 + a_2 + a_3
prefix_sum[3+1]-prefix_sum[1] # => 8
~~~

# Counting sort O(n+m)

~~~
a=[4,2,0,2,3]
c=[0] * (a.max+1)
a.each_with_index do |e,i|
  c[e] += 1
end
s=[]
i=0
c.each_with_index do |n,v| # => [1, 0, 2, 1, 1]
  n.times do
    s[i] = v
    i+=1
  end
end
s # => [0, 2, 2, 3, 4]
~~~

# Merge sort O(n*log(n))

~~~
a=[4,2,0,2,3]

def sort(a)

  if a.length == 1
    return a # => [4], [2], [0], [2], [3]
  else
    middle = a.length/2
    a[0..middle-1] # => [4, 2], [4], [0], [2]
    a[middle..-1] # => [0, 2, 3], [2], [2, 3], [3]
    left = sort(a[0..middle-1]) # => [4], [2, 4], [0], [2]
    right = sort(a[middle..-1]) # => [2], [3], [2, 3], [0, 2, 3]
    i = 0
    j = 0
    r = []
    while ! left[i].nil? || ! right[j].nil?
      if ! left[i].nil? && (right[j].nil? || left[i] <= right[j])
        r << left[i] # => [2, 4], [2], [0], [0, 2], [0, 2, 2, 3, 4]
        i += 1
      else
        r << right[j] # => [2], [2, 3], [0, 2], [0, 2, 3], [0], [0, 2, 2], [0, 2, 2, 3]
        j += 1
      end
    end
    return r # => [2, 4], [2, 3], [0, 2, 3], [0, 2, 2, 3, 4]
  end
end
sort(a) # => [0, 2, 2, 3, 4]
~~~

* leader (value on more than half elements) is the same when we remove pair of
  different items
* maximum slice is a slice (a_p..a_q) with max sum, can be computed in O(n^2)

  ~~~
  # max_slice O(n^2)
  a=[5,-7,3,5,-2,4,-1]
  res = 0
  a.each_with_index do |p,i|
    sum = 0
    a[i..-1].each do |q|
      sum += q
      res = [res,sum].max
    end
  end
  res # => 10
  ~~~

* positive ending sum slice is wrapped with negative elements (or bounds
  first/last) because we can always choose empty slice (sum is 0) we can deduce
  that positive_ending_sum = max(0, positive_ending_sum_prev + a)

  ~~~
  # max_slice O(n)
  a=[5,-7,3,5,-2,4,-1]

  res = 0
  positive_ending_sum = 0
  a.each do |p| # => [5, -7, 3, 5, -2, 4, -1]
    positive_ending_sum = [0, positive_ending_sum+ p].max # => 5, 0, 3, 8, 6, 10, 9
    res = [res,positive_ending_sum].max # => 5, 5, 5, 8, 8, 10, 10
  end
  res # => 10
  ~~~

# Prime Numbers

* count number of divisors of N is O(sqrt(N)) since every divisor a has symetric
  divisor b, `n = a * b`

~~~
n = 30
i = 1
result = 0
while i * i < n
  result += 2 if n % i == 0
  i += 1
end
result += 1 if i * i == n
~~~

* finding all primes less than N is in O(N*log(log(N))) Sieve of Eratosthenes.
Find first prime (until sqrt(N) since we do not need to mark greater than N) and
mark all multiplies of prime, starting from prime*prime (since i*prime will be
marked with prime divisor of i) untill N, that is n/2 + n/3+ n/5 ... =
n*log(log(n)
Rembember that largest prime factor of N can not be greated than Math.sqrt(N).
0, 1 are not prime numbers.

  ~~~
  m = 10
  definitely_non_prime = Array.new m + 1, false
  definitely_non_prime[0] = definitely_non_prime[1] = true
  2.upto(Math.sqrt(m)) do |i|
    next if definitely_non_prime[i]
    prime = i
    non_prime = prime * prime
    while non_prime <= m
      definitely_non_prime[non_prime] = true
      non_prime += prime
    end
  end
  definitely_non_prime.each_with_index { |e,i| puts i if e == false }
  ~~~

* factorization is to find prime numbers and its order. We can create fixed F
  where F[i] is smallest prime that divides i, ie F[12] = 2, F[6] = 2, F[2] = 0.
  If we already have that F, than we can factorize in O(log(n)) time

  ~~~
  n = 12
  f = [0]*(n+1)
  i=2
  while i*i <= n
    if f[i] == 0
      prime = i
      temp = prime * prime
      while temp <= n
        f[temp] = prime if f[temp] == 0
        temp += prime
      end
    end
    i += 1
  end
  f # => [0, 0, 0, 0, 2, 0, 2, 0, 2, 3, 2, 0, 2]

  def factorization(n, f)
    res = []
    while f[n] > 0
      res << f[n] # => [2], [2, 2]
      n /= f[n] # => 6, 3
    end
    res << n # => [2, 2, 3]
  end

  factorization(n,f) # => [2, 2, 3]
  ~~~

* we can find all divisors, for example 12 has [1,2,3,4,6,12] divisor list, with
  simply iterate to all numbers and add number (for example 2) to set for
  product (divisors[4].add(2), divisors[6].add(2) ...).  Better perfomance is
  when we stop at `number * number <= n` and we add
  `divisors[product].add(product/number)` (note that we still iterate to n in
  inner loop).

  ~~~
  n = 12
  divisors = [] * (n+1)
  [*1..n].each do |i| # => [1, 2, 3, 4, 5, 6]
    divisors[i] = Set.new([1,i]) # => #<Set: {1}>, #<Set: {1, 2}>, #<Set: {1, 3}>, #<Set: {1, 4}>, #<Set: {1, 5}>, #<Set: {1, 6}>
  end
  number = 2
  while number <= n  # number * number <= n
    product = number # => 2, 3, 4, 5, 6
    while product <= n
      if ! divisors[product].include? number
        divisors[product].add number # => #<Set: {1, 4, 2}>, #<Set: {1, 6, 2}>, #<Set: {1, 6, 2, 3}>
        # divisors[product].add(product/number)
      end
      product += number
    end
    number += 1
  end
  ~~~

# Euclidean algorithm

Theory:

Def: integer m divides n (m|n) if exists r such that m*r = n
Def: gcm(m, n) is the largest positive integer d which divides both m and n.
That means d|m, d|n and for any c which c|m and c|n we have d>=c

The: for a, b nonzero integer, their gcd(a,b) is linear combination of a and b
ie: exists s and t such that s*a+t*b=gcd(a,b)
Proof: let d be least positive linear combination d = s*a+t*b . We can show that
d|a.
We can write `a = d*q+r (0 <= r < d )` so `r = a - d*q = a - (s*a + t*b)*q = (1
- q*s)*a + (-qt)*b`. Since d is least positive and `r<d` that means `r = 0`.
Similarly d|b. We can show it is the greatest common divisor because if
d1=gcd(a,b) < d it will be that d1|a d1|b and from d=s*a+t*b that d1|d which is
contradiction so d1==d

gcd(k, 1) = 1
if a|b*c and gcd(a,b)==1 than a|c

Lem: if a = b*q + r than gcd(a,b) == gcd(b,r)
Proof: gcd(a,b)=gcd(b*q+r,b)=gcd(r,b)
GCD will not change if you can add multiple of b, ie gcd(a,b)=gcd(a+k*b,b)


Def: The integers a and b are relatively prime if and only if there exist
integers α and β such that aα + bβ = 1

Def: Lcm(a,b) least common multiple is = |a*b|/gcd(a,b)

The: a*x+b*y=d has integer solution iff gcd(a,b)|d

If (x0,y0) is solution, we can add +- a*b/d  ie a*x+b*y + a*b/d - b*a/d =
a*(x+b/d) +b(y-a/d)  so (x+ b/d, y-a/d) is solution.
General solutions are { (x0+k*b/d, y0-k*a/d) for k in Z}

* Found gcd(a,b) with Euclidean algorithm by subtraction in O(a+b) steps, or by
  division in O(log(a+b))

  ~~~
  def gcd(a,b)
    if a % b == 0
      return b
    else
      gcd(b, a % b)
    end
  end
  ~~~

* lcm(a,b) = a*b/gcd(a,b)  and lcm(a,b,c) = lcm(a, lcm(b,c))

* Fibonacci numbers F[n] = F[n-1] + F[n-2] and F[0] = 0 and F[1] = 1 and can be
  generated in O(N)
* faster could be with matrix calculation

* binary search for sorted array could be in O(log(N)) time. We found middle
  element and test condition.  Array should be sorted (ascending) so we can test
  (mid<=target) and pick next item. Result is last middle when condition was
  true (max element <= target). If it is true go with (bigger) elements,
  othervise go with lower elements. We do not stop when we found solution since
  usual we have optimization problem (min or max). When we pass solution and set
  first = middle +1 (could be equal last) next iteration will be false (or last
  = new_middle -1 until last < first).  If we search for min, then if condition
  is true we took lower elements (last = mid - 1). That is easy to replace in
  check function but result should be taken from that as well. So general check
  function store result if test is true with max, or test is false with min.
  Another nice non general solution is to return last for max, or first for min.

  ~~~
  a=[1,2,3,4,5] #[12,15,15,19,24,31,53,59,60,61]
  target = 3 # 13 
  def check(a, middle, target, result)
    if a[middle] <= target
      result.value = middle
      true
    else
      false
    end
  end
  #a = [0,1,1,0,1,1,0,1,1]
  #target = 3 # k boards 3 boards
  #def check(a, middle, target, result)
  #  last_board = -1
  #  boards = 0
  #  middle # => 4, 1, 2
  #  for i in 1..a.length
  #    if a[i] == 1 && last_board < i
  #      boards += 1 # => 1, 2, 1, 2, 3, 4, 5, 6, 1, 2, 3
  #      last_board = i + middle - 1 # => 4, 8, 1, 2, 4, 5, 7, 8, 2, 5, 8
  #    end
  #  end
  #  boards # => 2, 6, 3
  #  if boards <= target # => true, false, true
  #    # since we search to min, we invert return value
  #    result.value = middle
  #    false
  #  else
  #    true
  #  end
  #end
  class Result
    attr_accessor :value
  end
  def binary_search(a, target)
    first = 0
    last = a.length - 1
    result = Result.new
    result.value = -1
    while first <= last
      first     # => 0, 3
      last    # => 4, 4
      middle = (first+last)/2 # => 2, 3
      if check(a, middle, target, result) # => true, false
        first = middle + 1 # => 3
      else
        last = middle - 1 # => 2
      end
    end
    return result.value
  end
  binary_search(a, target) # => 2
  ~~~

* caterpillar method. found subseqence which total sum equals s

  ~~~
  a = [6,2,7,4,1,3,6]
  target_sum = 12

  back = 0
  front = 0
  sum = 0
  a.each_with_index do |e, back|
    while front < a.length && sum + a[front] <= target_suM
      sum += a[front]
      front += 1
    end
    if sum == target_sum
      puts "je"
    end
    sum -= e
  end
  ~~~

* greedy algorithm is simplest possible algorithm
* dinamic programming

  ~~~
  def dynamic_coin_changing(c, k)
    n = c.length
    dp = [0] + [1/0.0]* k # => [0, Infinity, Infinity, Infinity, Infinity, Infinity, Infinity]
    for i in 1..n # => 1..3
      for j in c[i-1] .. k # => 1..6, 3..6, 4..6
         # => 1..6, 3..6, 4..6
        dp[j-c[i-1]]+1 # => 1, 2, 3, 4, 5, 6, 1, 2, 3, 2, 1, 2, 3
  dp[j] # => Infinity, Infinity, Infinity, Infinity, Infinity, Infinity, 3, 4, 5, 6, 2, 3, 2
        dp[j] = [dp[j-c[i-1]]+1, dp[j]].min # => 1, 2, 3, 4, 5, 6, 1, 2, 3, 2, 1, 2, 2
      end
      dp # => [0, 1, 2, 3, 4, 5, 6], [0, 1, 2, 1, 2, 3, 2], [0, 1, 2, 1, 1, 2, 2]
    end
    dp
  end
  dynamic_coin_changing([1,3,4],6) # => [0, 1, 2, 1, 1, 2, 2]
  ~~~

# Geometry

* Find line that is perpendicular to line `y = 2*x + 11` and contains `(6, -7)`
  [video](https://www.khanacademy.org/math/geometry/hs-geo-analytic-geometry/hs-geo-parallel-perpendicular-eq/e/line_relationships)
  Perpendicular means slope is negative inverse ie `-1/2`. Note that in ruby you
  need to use `x = x.to_f` since `1/2==0` for integer division
* maximum area for fixed length sides (edges) polygon has edges on circle
  (descripted circuit) and orders does not matter
  [link](http://www.drking.org.uk/hexagons/misc/polymax.html)
* triangle area `s = (a+b+c)/2; P = Math.sqrt(s*(s-a)*(s-b)*(s-c));`

# Math

* `radius = Math.sqrt((l1/2)**2 + (l2/2)**2)`

# Tutorial

* <https://www.toptal.com/algorithms/interview-questions>

# Constrains

* memory: usually if you need to loop more than 10**6 (1MB) that could be a
problem
