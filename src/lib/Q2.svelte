<!-- This code is modified based on AI-generated code by Danhua Zhang 02/04/2025-->

<script lang="ts">
  import type { TMovie } from "../types";
  import * as d3 from "d3";
  import { onMount } from "svelte";

  type Props = {
    movies: TMovie[];
    progress?: number;
    width?: number;
    height?: number;
  };

  let { movies, progress = 100, width = 500, height = 500 }: Props = $props();

  function getGenreCoocurrances(movies: TMovie[]) {
    let genreCoocurrances = new Map<string, Map<string, number>>();
    movies.forEach((movie) => {
      movie.genres.forEach((genre1) => {
        movie.genres.forEach((genre2) => {
          if (genre1 !== genre2) {
            if (!genreCoocurrances.has(genre1)) {
              genreCoocurrances.set(genre1, new Map<string, number>());
            }
            const genre1Map = genreCoocurrances.get(genre1);
            genre1Map.set(genre2, (genre1Map.get(genre2) || 0) + 1);
          }
        });
      });
    });
    return genreCoocurrances;
  }
  let genreCoocurrances = $derived(getGenreCoocurrances(movies));

  const margin = {
    top: 15,
    bottom: 50,
    left: 30,
    right: 10,
  };

  let usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left,
  };

  let svg;

  $effect(() => {
    drawChordDiagram();
  });

  function drawChordDiagram() {
    
    const genres = Array.from(genreCoocurrances.keys());
    const matrix = genres.map((genre1) =>
      genres.map((genre2) => genreCoocurrances.get(genre1).get(genre2) || 0)
    );
    console.log(genreCoocurrances);
    const chord = d3.chord().padAngle(0.05).sortSubgroups(d3.descending)(matrix);

    const arc = d3.arc().innerRadius(width / 2 - 100).outerRadius(width / 2 - 80);

    const ribbon = d3.ribbon().radius(width / 2 - 100);

    const color = d3.scaleOrdinal(d3.schemeCategory10).domain(genres);

    const svgElement = d3
      .select(svg)
      .attr("width", width)
      .attr("height", height)
      .append("g")
      .attr("transform", `translate(${width / 2},${height / 2})`);

    const gradient = svgElement.append("defs")
      .append("linearGradient")
      .attr("id", "arcGradient")
      .attr("x1", "0%")
      .attr("y1", "0%")
      .attr("x2", "100%")
      .attr("y2", "100%");

    gradient.append("stop")
      .attr("offset", "0%")
      .attr("stop-color", "blue");

    gradient.append("stop")
      .attr("offset", "100%")
      .attr("stop-color", "red");

    const tooltip = d3.select("body").append("div")
      .attr("class", "tooltip")
      .style("position", "absolute")
      .style("visibility", "hidden")
      .style("background", "#fff")
      .style("border", "1px solid #ccc")
      .style("padding", "5px")
      .style("border-radius", "5px");

    svgElement
      .datum(chord)
      .append("g")
      .selectAll("path")
      .data((d) => d.groups)
      .enter()
      .append("path")
      .style("fill", (d) => color(genres[d.index]))
      .style("stroke", (d) => d3.rgb(color(genres[d.index])).darker())
      .attr("d", arc)
      .on("mouseover", function (event, d) {
        d3.select(this).style("fill", "yellow").raise();
        tooltip.style("visibility", "visible")
          .text(`${genres[d.index]}: ${d.value}`);
        
      })
      .on("mousemove", function (event) {
        tooltip.style("top", (event.pageY - 10) + "px")
          .style("left", (event.pageX + 10) + "px");
      })
      .on("mouseout", function () {
        d3.select(this).style("fill", (d) => color(genres[d.index]));
        tooltip.style("visibility", "hidden");
      });

    svgElement
      .datum(chord)
      .append("g")
      .selectAll("path")
      .data((d) => d)
      .enter()
      .append("path")
      .attr("d", ribbon)
      .style("fill", function (d) {
        const radius = width / 2 - 100; // Use the same radius as your chord layout
        const x1 = Math.cos((d.source.startAngle + d.source.endAngle) / 2 - Math.PI / 2) * radius;
        const y1 = Math.sin((d.source.startAngle + d.source.endAngle) / 2 - Math.PI / 2) * radius;
        const x2 = Math.cos((d.target.startAngle + d.target.endAngle) / 2 - Math.PI / 2) * radius;
        const y2 = Math.sin((d.target.startAngle + d.target.endAngle) / 2 - Math.PI / 2) * radius;

        const gradientId = `gradient-${d.source.index}-${d.target.index}`;
        const gradient = svgElement.append("defs")
        .append("linearGradient")
        .attr("id", gradientId)
        .attr("gradientUnits", "userSpaceOnUse")
        .attr("x1", x1)
        .attr("y1", y1)
        .attr("x2", x2)
        .attr("y2", y2);

        gradient.append("stop")
          .attr("offset", "0%")
          .attr("stop-color", color(genres[d.target.index]));

        gradient.append("stop")
          .attr("offset", "100%")
          .attr("stop-color", color(genres[d.source.index]));

        return `url(#${gradientId})`;
      })
      //.style("stroke", (d) => d3.rgb(color(genres[d.target.index])).darker())
      .on("mouseover", function (event, d) {
        d3.select(this).style("fill", "yellow").raise();
        tooltip.style("visibility", "visible")
          .text(`${genres[d.source.index]} & ${genres[d.target.index]}: ${d.source.value}`);
        console.log(d.source.index, d.target.index, d.source.startAngle, d.target.startAngle);
        console.log("correct source", genres[d.source.index]);
        console.log("correct target", genres[d.target.index]);
      })
      .on("mousemove", function (event) {
        tooltip.style("top", (event.pageY - 10) + "px")
          .style("left", (event.pageX + 10) + "px");
      })
      .on("mouseout", function () {
        d3.select(this).style("fill", function (d) {
          const gradientId = `gradient-${d.source.index}-${d.target.index}`;
          return `url(#${gradientId})`;
        });
        tooltip.style("visibility", "hidden");
      });

    // Add genre labels
    svgElement
      .datum(chord)
      .append("g")
      .selectAll("text")
      .data((d) => d.groups)
      .enter()
      .append("text")
      .each(function (d) {
        d.angle = (d.startAngle + d.endAngle) / 2;
        d.name = genres[d.index];
      })
      .attr("dy", ".35em")
      .attr("transform", function (d) {
        return (
          "rotate(" +
          ((d.angle * 180) / Math.PI - 90) +
          ")" +
          "translate(" +
          (width / 2 - 70) +
          ")" +
          (d.angle > Math.PI ? "rotate(180)" : "")
        );
      })
      .style("text-anchor", function (d) {
        return d.angle > Math.PI ? "end" : null;
      })
      .text(function (d) {
        return d.name;
      });
  }
</script>

<h3>
  Q2: Are there any correlations between different genres?
</h3>

{#if movies.length > 0}
  <svg bind:this={svg} {width} {height}></svg>
  
{/if}

<style>
  .tooltip {
    font-size: 12px;
  }
</style>