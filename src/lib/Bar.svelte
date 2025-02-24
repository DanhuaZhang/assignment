<script lang="ts">
  import type { TMovie } from "../types";
  import * as d3 from "d3";
  // define the props of the Bar component
  type Props = {
    movies: TMovie[];
    progress?: number;
    width?: number;
    height?: number;
  };
  // progress is 100 by default unless specified
  let { movies, progress = 100, width = 500, height = 400 }: Props = $props();

  let selectedGenre: string = $state("");

  // processing the data; $derived is used to create a reactive variable that updates whenever the dependent variables change
  const yearRange = $derived(d3.extent(movies.map((d) => d.year)));

  function getUpYear(yearRange: [undefined, undefined] | [Date, Date]) {
    if (!yearRange[0]) return new Date();
    const timeScale = d3.scaleTime().domain(yearRange).range([0, 100]);
    return timeScale.invert(progress);
  }
  const upYear: Date = $derived(getUpYear(yearRange!));

  function getGenreNums(movies: TMovie[], upYear: Date) {
    let res: { [genre: string]: number } = {};
    movies
      .filter((movie) => movie.year <= upYear)
      .forEach((movie) => {
        movie.genres.forEach((genre: string) => {
          res[genre] = (res[genre] || 0) + 1;
        });
      });
    return res;
  }

  const genreNums = $derived(getGenreNums(movies, upYear));

  // drawing the bar chart

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

  const xScale = $derived(
    // Create a band scale for the x-axis using the genre names as the domain
    d3
      .scaleBand()
      .domain(Object.keys(genreNums))
      .range([usableArea.left, usableArea.right])
      .padding(0.2)
  );

  const yScale = $derived(
    // Create a linear scale for the y-axis using the count values as the domain
    d3
      .scaleLinear()
      .domain([0, d3.max(Object.values(genreNums)) || 0])
      .range([height - margin.bottom, margin.top])
  );

  const xBarwidth: number = $derived(xScale.bandwidth());

  let xAxis: any = $state(),
    yAxis: any = $state();

  function updateAxis() {
    d3.select(xAxis)
      .call(d3.axisBottom(xScale))
      .selectAll("text")
      .attr("transform", "rotate(45)")
      .style("text-anchor", "start");

    d3.select(yAxis).call(d3.axisLeft(yScale))
      .selectAll("text")
      .attr("transform", "rotate(-30)")
      .style("text-anchor", "end");
  }

  // the $effect function is used to run a function whenever the reactive variables change, also known as a side effect
  $effect(() => {
    updateAxis();
  });
</script>

<h3>
  The Distribution of Genres {yearRange[0]?.getFullYear()} - {yearRange[1]?.getFullYear()}
</h3>

{#if movies.length > 0}
  <svg {width} {height}>
    <g class="bars">
      {#each Object.entries(genreNums) as [genre, cnt]}
        <g class={genre}>
          <!-- Drawing bars using the SVG <rect/> component -->
          <rect
            width={xBarwidth}  
            height={yScale(0) - yScale(cnt)}  
            x={xScale(genre)} 
            y={yScale(cnt)} 
            fill="#449900"
            class="bar"
            opacity={selectedGenre === genre ? 1 : 0.5}  
            onmouseover={() => {
              selectedGenre = genre; // Store the hovered genre
            }}
            onmouseout={() => {
              selectedGenre = ""; // Clear the selection on mouse out
            }}
          />

          <!-- Displaying the count of movies on top of each bar -->
          <text
            x={xScale(genre)! + xBarwidth / 2}  
            y={yScale(cnt) - 5}  
            font-size="12"
            text-anchor="middle"
          >
          {#if selectedGenre === genre}
            {genre}: {cnt}
          {:else}
            {cnt}
          {/if}
          </text>
        </g>
      {/each}
    </g>

    <!-- Adding X and Y axes -->
    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
  </svg>
{/if}