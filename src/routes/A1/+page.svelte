<script lang="ts">
    import * as d3 from "d3";
    import { onMount } from "svelte";
    import type { TMovie } from "./src/types";
    import Bar from "$lib/Bar.svelte";
    import Q1 from "$lib/Q1.svelte";
    import Q2 from "$lib/Q2.svelte";
  
    // Reactive variable for storing the data
    let movies: TMovie[] = [];


    // Function to load the CSV
    async function loadCsv() {
      try {
        const csvUrl = "./summer_movies.csv";
        movies = await d3.csv(csvUrl, (row) => {
          // TIP: in row, all values are strings, so we need to use a row conversion function here to format them
          return {
            tconst: row.tconst,
            title_type: row.title_type,
            primary_title: row.primary_title,
            original_title: row.original_title,
            year: new Date(row.year),
            runtime_minutes: Number(row.runtime_minutes),
            genres: row.genres.split(","),
            simple_title: row.simple_title,
            average_rating: Number(row.average_rating),
            num_votes: Number(row.num_votes),
          };
        });
  
        console.log("Loaded CSV Data:", movies);
      } catch (error) {
        console.error("Error loading CSV:", error);
      }
    }
    // Call the loader when the component mounts
    onMount(loadCsv);
  </script>
  
  <h1>Summer Movies</h1>
  
  <p>Here are {movies.length == 0 ? "..." : movies.length + " "} movies</p>
  <Bar {movies} />

  <Q1 {movies} />

  <Q2 {movies} />
  