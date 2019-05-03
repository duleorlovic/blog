---
layout: post
---

# D3

https://d3js.org/

Use declarative approach like jQuery `d3.selectAll('p').style('color', 'blue')`.
Instead of static values, properties can be specified as functions of data (for
example path data). You can pass data `.data([1,2])` to bound to a selection. If
there are fewer nodes than data, extra data elements form the `enter` selection
`.enter().append('p')`
You can add `.transition().duration(500).delay(function(d,i) { return i*10})`

Nice tutorial for bar chart using svg
https://alignedleft.com/tutorials/d3/making-a-bar-chart

```
var data = [100,400,200, 220, 433, 123, 1, 333]
var w = 500
var h = Math.max(...data)
var barPadding = 1
var font_size = 16

var svg = d3.select('.chart')
  .append('svg')
  .attr('width', w)
  .attr('height', h)

svg.selectAll('rect')
  .data(data)
  .enter()
  .append('rect')
  .attr('x', (d, i) => i * (w/data.length) )
  .attr('y', (d) => h - d )
  .attr('width', w / data.length - barPadding)
  .attr('height', (d) => d )
  .attr('fill', (d) => 'rgb(0,0,' + (d*255/h) + ')' )

svg.selectAll('text')
  .data(data)
  .enter()
  .append('text')
  .text((d) => d)
  .attr('text-anchor', 'middle')
  .attr('x', (d, i) => i * (w/data.length) + (w/data.length - barPadding)/2 )
  .attr('font-size', font_size)
  .attr('font-family', 'sans-serif')
  .attr('y', (d) => d > font_size ? h + font_size - d : h - 2 )
  .attr('fill', (d) => d > font_size ? 'white' : 'black' )
```
