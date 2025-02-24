<!-- This code is modified based on AI-generated code by Danhua Zhang 02/04/2025-->

<script lang="ts">
  import type { TMovie } from "../types";
  import * as d3 from "d3";
  
  // define the props of the heatmap component
  type Props = {
    movies: TMovie[];
    width?: number;
    height?: number;
  };
  
  // progress is 100 by default unless specified
  let { movies, width = 1200, height = 500 }: Props = $props();

  type GenreCount = {
    [genre: string]: number;
  };

  type YearGenreCount = {
    [year: string]: GenreCount;
  };

  let selectedGenre: string = $state("");
  let selectedYear: number = $state(0);
  let selectedCount: number = $state(0);
  let colorIndex: number = $state(0);

  const margin = {
    top: 15,
    bottom: 50,
    left: 100,
    right: 10,
  };

  let usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left,
  };

  const yearRange = $derived(d3.extent(movies.map((d) => d.year).sort()));
  // console.log("Year Range:", yearRange);

  // Function to get the top 3 genres per year using Record
  function getGenreCountPerYear(movies: TMovie[]): { year: number, genres: [string, number][] }[] {
    let genreCountPerYear: YearGenreCount = {};

    // Compute the genre count per year
    movies.forEach(movie => {
      const year = movie.year.getFullYear().toString();
      if (!genreCountPerYear[year]) {
        genreCountPerYear[year] = {};
      }
      movie.genres.forEach(genre => {
        genreCountPerYear[year][genre] = (genreCountPerYear[year][genre] || 0) + 1;
      });
    });

    // Sort years and genres
    return Object.keys(genreCountPerYear)
      .sort((a, b) => Number(a) - Number(b)) // Sort years numerically
      .map(year => {
        const genres = genreCountPerYear[year];

        // Sort genres by their count values in descending order
        const sortedGenres: [string, number][] = Object.entries(genres)
          .sort(([, countA], [, countB]) => countB - countA); // Descending sort by count

        return {
          year: Number(year),
          genres: sortedGenres
        };
      });
  }

  // Using $derived to update sortedGenresPerYear when movies change
  const sortedGenresPerYear = $derived(getGenreCountPerYear(movies));
  console.log("Sorted Genres Per Year:", sortedGenresPerYear);

  // Define the color scale based on genre types
  const genreList = $derived(Array.from(new Set(movies.flatMap(movie => movie.genres))));
  const colorScale = $derived(d3.scaleOrdinal(d3.schemeCategory10).domain(genreList));
  
  // Define the color scale for the top 3 genres
  const topColorScale = ["red", "orange", "yellow"];
    
  let xAxis: any = $state(),
    yAxis: any = $state();

  const xScale = $derived(
    d3
      .scaleBand()
      .domain(sortedGenresPerYear.map(entry => entry.year.toString()))
      .range([usableArea.left, usableArea.right])
      .padding(0.2)
  );

  const yScale = $derived(
    d3
      .scaleBand()
      .domain(genreList)
      .range([height - margin.bottom, margin.top])
      .padding(0.2)
  );

  function updateAxis() {
    // console.log("Sorted Genres Per Year:", sortedGenresPerYear);

    d3.select(xAxis)
      .call(d3.axisBottom(xScale))
      .selectAll("text")
      .attr("transform", "rotate(45)")
      .style("text-anchor", "start");

    d3.select(yAxis).call(d3.axisLeft(yScale))
      .selectAll("text")
      .attr("transform", "rotate(0)")
      .style("text-anchor", "end");
  }

  // the $effect function is used to run a function whenever the reactive variables change, also known as a side effect
  $effect(() => {
    updateAxis();
  });

</script>

<h3>
  Q1: How do the top three movie genres (by number of movies) change over time?
</h3>

{#if movies.length > 0}
  <svg width={width} height={height}>
    {#each sortedGenresPerYear as { year, genres }}
      {#each genres as [genre, count], index}
        <!-- svelte-ignore a11y_no_static_element_interactions -->
        <!-- svelte-ignore a11y_mouse_events_have_key_events -->
        <rect
          x={xScale(year.toString())}
          y={yScale(genre.toString())}
          width={xScale.bandwidth()}
          height={yScale.bandwidth()}
          fill={index < 3 ? topColorScale[index] : "gray"}
          opacity={index < 3 ? 1 : 0.2}
          onmouseover={() => {
            selectedGenre = genre;
            selectedYear = year;
            selectedCount = count;
          }}
          onmouseout={() => {
            selectedGenre = "";
            selectedYear = 0;
            selectedCount = 0;
          }}
          
        />
      {/each}
    {/each}

    {#each sortedGenresPerYear as { year, genres }}
      {#each genres as [genre, count]}
        {#if selectedGenre === genre && selectedYear === year}
          <!-- Background Box -->
          <g pointer-events="none"> <!-- Group to avoid pointer events on text and box -->
            <rect
              x={xScale(year.toString()) + xScale.bandwidth() / 2 - 60} 
              y={yScale(genre) - 55} 
              width="120" 
              height="50"
              fill="white"
              stroke="black" 
              stroke-width="1"
              rx="5" 
            />
            
            <!-- Tooltip Text -->
            <text
              x={xScale(year.toString()) + xScale.bandwidth() / 2}
              y={yScale(genre) - 40}
              font-size="12"
              text-anchor="middle"
              fill="black"
              pointer-events="none"
              style="user-select: none;"
            >
              <tspan>Year: {selectedYear}</tspan>
              <tspan x={xScale(year.toString()) + xScale.bandwidth() / 2} dy="1.2em">Genre: {genre}</tspan>
              <tspan x={xScale(year.toString()) + xScale.bandwidth() / 2} dy="1.2em">Count: {selectedCount}</tspan>
            </text>
          </g>
        {/if}
    
      {/each}
    {/each}

    <!-- Adding X and Y axes -->
    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />

    <!-- Adding Legend -->
    <g transform="translate({margin.left + 10}, {margin.top})">
      {#each topColorScale as color, index}
        <rect
          x="0"
          y={index * 20}
          width="18"
          height="18"
          fill={color}
        />
        <text
          x="24"
          y={index * 20 + 9}
          dy=".35em"
          font-size="12"
          text-anchor="start"
          fill="black"
        >
          Top {index + 1} Genre
        </text>
      {/each}
      <rect
        x="0"
        y={topColorScale.length * 20}
        width="18"
        height="18"
        fill="gray"
        opacity="0.2"
      />
      <text
        x="24"
        y={topColorScale.length * 20 + 9}
        dy=".35em"
        font-size="12"
        text-anchor="start"
        fill="black"
      >
        Other Genres
      </text>
    </g>
  </svg>
{/if}