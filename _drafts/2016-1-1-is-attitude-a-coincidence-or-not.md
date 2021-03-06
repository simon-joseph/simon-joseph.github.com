---
title: Attitude - coincidence or not?
excerpt: "... and it all started with this picture I saw."
header:
    teaser: "https://farm2.staticflickr.com/1558/23945710782_d757fa6fa7_o.jpg"
date: 2016-1-1
modified: 2016-1-1
tags:
    - python
    - fun
    - statistics
d3js: true
chartjs: true
visjs: true
---

{% include toc icon="columns" title="Overview" %}

## The picture and some fun facts

<figure class="grayscale-image-wrap">
	<a href="https://farm2.staticflickr.com/1558/23945710782_d757fa6fa7_o.jpg"><img src="https://farm2.staticflickr.com/1558/23945710782_e0d97e2a38_b.jpg"></a>
	<figcaption>
        Picture as seen on a LinkedIn post. See on
        <a href="https://www.flickr.com/photos/126937514@N03/23945710782/in/album-72157662960145625/" target="blank" title="Picture as seen on a LinkedIn post.">
            <b>Flickr.com</b>
        </a>
    </figcaption>
</figure>

Take a pause here and give yourself a moment to think about what's going on in this picture.

> Done? Ok, you had a chance for an unbiased opinion.

I saw this image on LinkedIn last week and was much amused. Part of me
understands it is a memorable way to highlight the importance of _attitude_
at work. The other part could not help to make fun of this. Translating words
to a percentage of _importance at work_ is plain nonsense. Some words even map
to more than 100%, e.g. `nonsense` translates to `105`. Still, it's easy to
ridicule things, but there's little value to it, so I decided to investigate
a bit further.

## What are the odds?

`words` is a standard file on all Unix-like OS, found in
`/usr/share/dict/words`. It contains a newline-delimited list of 235886
dictionary words. I'll assume it is a representative sample. With this, we can
easily compute that there are exactly `2302` words in there that translate
to `100%`.

    abactinally
    abrogative
    absinthol
    acclimation
    accumulate
    acenaphthene
    achroous
    acknowledge
    acriflavine
    acromegaly
    ...

`immature`, `grumpy`, `unclever` and `foolship` are just a few of the ones that
I found quite funny in the context of the picture in question.  
But what are the odds? Let us assume all these words ended up in `100% bucket` -
it contains about `0.98%` of total words. What about other buckets?
Here is what we get if we check the count of all possible buckets.

<center id="chart01"></center>

Data fits normal distribution with parameters:

    mean               = 112.3882
    standard deviation = 40.89031

With this, we see that it's not that uncommon to end up with a value near `112`.
Actually, if we permute the way we assign numbers to letters we will end up with
a normal distribution as well. This might seems surprising at first, but it
makes more sense when you have a look at the distribution of word lengths.

<center id="chart02"></center>

Let us have a look at the distribution of counts with regard to value grouped by
length.

<center id="chart03"></center>

Distribution in red reveals that it's not that uncommon for a word of length `8`
to have a value of `100`.

## How these statistics look for Polish

Gradually I lost further interest in answering the initial question. Instead, I
wanted to check if Polish set of words would have similar qualities. For
comparison, I used a set of polish dictionary words available
[here](http://sjp.pl/slownik/growy/) converted from `win-1250` to `utf-8` for
convenience.


    iconv -f CP1250 -t UTF8 slowa-win.txt > slowa-utf8.txt

First let us check how does the words length

<center id="chart04"></center>
<br>
<br>
<center id="chart05"></center>
<br>
<br>
<center id="chart06"></center>
<br>
<br>
<center id="chart07"></center>

<script>
Chart.defaults.global.legend.display = false;
appendCanvas(
    "chart01",
    "Count of words grouped by value calculated from the given formula.",
    function(title, holder, canvas, ctx) {
        d3.tsv("/assets/dsv/0001-value-count-stats-en.tsv", function(tsv) {
            var data = {
                labels : [],
                datasets: [
                    {
                        label       : 'Count: ',
                        fillColor   : "rgba(220,220,220,0.2)",
                        strokeColor : "rgba(220,220,220,1)",
                        pointColor  : "rgba(220,220,220,1)",
                        pointStrokeColor     : "#fff",
                        pointHighlightFill   : "#fff",
                        pointHighlightStroke : "rgba(220,220,220,1)",
                        data : []
                    }
                ]
                };
            var options = {
                scales: {
                    xAxes: [{
                        ticks: {
                            beginAtZero: true,
                            autoSkip: true,
                            autoSkipPadding: 23,
                            maxRotation: 0
                        }
                    }]
                }
            };
            data.labels.push(0);
            data.datasets[0].data.push(0);
            tsv.forEach(function(d) {
                data.labels.push(d.Value);
                data.datasets[0].data.push(+d.Count);
            });
            new Chart(ctx, { type: 'bar', data: data, options: options });
        });
    });

appendCanvas(
    "chart02",
    "Words length distribution.",
    function(title, holder, canvas, ctx) {
        d3.csv("/assets/dsv/0002-word_length-count-stats-en.csv", function(csv) {

            var data = {
                labels : [],
                datasets: [
                    {
                        label       : title,
                        fillColor   : "rgba(220,220,220,0.8)",
                        strokeColor : "rgba(220,220,220,1)",
                        pointColor  : "rgba(220,220,220,1)",
                        pointStrokeColor     : "#fff",
                        pointHighlightFill   : "#fff",
                        pointHighlightStroke : "rgba(220,220,220,1)",
                        data : []
                    }
                ]
                };

            csv.forEach(function(d) {
                data.labels.push(d["Word Length"]);
                data.datasets[0].data.push(+d.Count);
            });
            new Chart(ctx, { type: 'bar', data: data});
        });
    });

appendCanvas(
    "chart03",
    "Word count for bucket - chart per word length.",
    function(title, holder, canvas, ctx) {
        d3.csv("/assets/dsv/0003-words_length-value-count-stats-en.csv", function(csv) {

            var data = {
                labels : [],
                datasets: []
                };

            var options = {
                responsive : true,
                showTooltips : false,
                scales: {
                    xAxes: [{
                        ticks: {
                            beginAtZero: true,
                            autoSkip: true,
                            autoSkipPadding: 23,
                            maxRotation: 0
                        }
                    }]
                }
            };

            var columns = []

            for(i = 1; i < 24; i++){
                var c = Math.floor((i/24.0) * 255.0);
                var color = "rgba(" + c + "," + c + "," + c + ",0.8)";
                if (i == 8 ){
                    color = "rgba(200, 0, 0, 0.3)"
                }
                data.datasets[i-1] =
                    {
                    type: 'line',
                    label       : "Words of length " + i,
                    strokeColor : "rgba(0,0,0,.5)",
                    fill: true,
                    backgroundColor : color,
                    pointRadius: 0,
                    data : []
                    }
                columns[i-1] = "Col" + i
            };

            for (i = 0; i < 264; i++) {
                data.labels.push(i);
            }

            csv.forEach(function(d) {
                for(i = 0; i < 23; i++){
                    data.datasets[i].data.push(+d[columns[i]]);
                }
            });
            new Chart(ctx, { type: 'line', data: data, options: options });
        });
    });

appendCanvas(
    "chart04",
    "Words Length Distribution in Polish word set.",
    function(title, holder, canvas, ctx) {
        d3.csv("/assets/dsv/0004-words_length-count-stats-pl.csv", function(csv) {

            var data = {
                labels : [],
                datasets: [
                    {
                        label       : title,
                        fillColor   : "rgba(220,220,220,0.8)",
                        strokeColor : "rgba(220,220,220,1)",
                        pointColor  : "rgba(220,220,220,1)",
                        pointStrokeColor     : "#fff",
                        pointHighlightFill   : "#fff",
                        pointHighlightStroke : "rgba(220,220,220,1)",
                        data : []
                    }
                ]
                };

            var options = {
                responsive : true,
                barStrokeWidth : 1,
                barValueSpacing : 3,
                barDatasetSpacing : 0
            };

            csv.forEach(function(d) {
                data.labels.push(d.Length);
                data.datasets[0].data.push(+d.Count);
            });
            new Chart(ctx, { type: 'bar', data: data, options: options });
        });
    });

appendCanvas(
    "chart05",
    "Comparison of character count in PL and EN word sets.",
    function(title, holder, canvas, ctx) {
        d3.tsv("/assets/dsv/0005-char-stat-pl-en.tsv", function(tsv) {
            canvas.style({heigth: "500px"})
            var data = {
                labels : [],
                datasets: [
                    {
                        label       : "Polish",
                        fillColor: "rgba(151,187,205,0.5)",
                        strokeColor: "rgba(151,187,205,.4)",
                        data : []
                    },
                    {
                        label       : "English",
                        fillColor: "rgba(240,220,220,0.5)",
                        strokeColor: "rgba(220,220,220,.4)",
                        data : []
                    }
                ]
                };

            var options = {
                responsive : true,
                showTooltips: false,
                scaleOverride: true,
                scaleSteps: 6,
                scaleStepWidth: 0.02,
                scaleStartValue: 0.0,
                pointDot: false,
                scaleIntegersOnly: false,
                scaleShowLabels: true,
                scaleLabel: "<%=value*100%>\%"
            };

            tsv.forEach(function(d) {
                procPL = +d.procPL;
                procEN = +d.procEN;
                threshold = .04;
                if(procPL > threshold || procEN > threshold){
                    data.labels.push(d.char);
                    data.datasets[0].data.push(procPL);
                    data.datasets[1].data.push(procEN);
                }
            });

            var chart = new Chart(ctx, { type: 'radar', data: data, options: options });
            var legend = chart.generateLegend();
            var em = holder.select("em");
            em.append("br")
            em.append("span").text("Only characters occuring > 5%.");
            em.append("div").classed("chart-legend", true).html(legend);
            em.selectAll("li").style({display: "inline-block", padding: "5px"});
        });
    });

appendCanvas(
    "chart06",
    "Words Value Distribution in Polish word set.",
    function(title, holder, canvas, ctx) {
        d3.csv("/assets/dsv/0006-value-count-stats-pl.csv", function(csv) {

            var data = {
                labels : [],
                datasets: [
                    {
                        label       : title,
                        fillColor   : "rgba(220,220,220,0.8)",
                        strokeColor : "rgba(220,220,220,1)",
                        pointColor  : "rgba(220,220,220,1)",
                        pointStrokeColor     : "#fff",
                        pointHighlightFill   : "#fff",
                        pointHighlightStroke : "rgba(220,220,220,1)",
                        data : []
                    }
                ]
                };

            var options = {
                responsive : true,
                barStrokeWidth : 1,
                barValueSpacing : 0,
                barDatasetSpacing : 0
            };

            csv.forEach(function(d) {
                if (+d.Value % 30 == 0) {
                    data.labels.push(d.Value);
                } else {
                    data.labels.push("");
                }
                data.datasets[0].data.push(+d.Count);
            });
            new Chart(ctx, { type: 'bar', data: data, options: options });
        });
    });

appendCanvas(
    "chart07",
    "Words Length/Value/Count in Polish word set.",
    function(title, holder, canvas, ctx) {
        d3.csv("/assets/dsv/0007-words_length-value-count-stats-pl.csv", function(csv) {

            var data = {
                labels : [],
                datasets: []
                };

            var options = {
                responsive : true,
                showTooltips : false,
                pointDot : false,
                datasetStroke : true,
                datasetStrokeWidth : 1,
                datasetFill : true
            };

            var columns = []

            for(i = 1; i < 16; i++){
                var c = Math.floor(((i % 16 )/19.0) * 255.0);
                var color = "rgba(" + c + "," + c + "," + c + ",1.0)";
                data.datasets[i-1] =
                    {
                    label       : "Words of length " + i,
                    strokeColor : "rgba(0,0,0,.5)",
                    fillColor : color,
                    data : []
                    }
                columns[i-1] = "Col" + i
            };

            var value = 0;
            csv.forEach(function(d) {
                value++;
                if (value % 15 == 0){
                    data.labels.push(value);
                } else {
                    data.labels.push("");
                }
                for(i = 0; i < 15; i++){
                    data.datasets[i].data.push(+d[columns[i]]);
                }
            });
            new Chart(ctx, { type: 'bar', data: data, options: options });
        });
    });
</script>

*[Polish]: It is my native language.
