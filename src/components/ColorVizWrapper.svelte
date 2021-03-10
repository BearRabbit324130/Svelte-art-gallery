<script>
  import * as d3 from "d3";
  import { onMount } from "svelte";
  import "intersection-observer";
  import scrollama from "scrollama";
  import mapToArray from "../utils/mapToArray";
  import ColorViz from "./ColorViz.svelte";
  export let data;

  // SCROLL!
  onMount(() => {
    // instantiate the scrollama
    const scroller = scrollama();

    // setup the instance, pass callback functions
    scroller
      .setup({
        step: "#colorSection .step",
      })
      .onStepEnter((response) => {
        activeStep = response.index;
        if (response.index == 0) {
          init();
        }
        if (response.index == 1) {
          createTimeline();
        }
        if (response.index == 2) {
          highlight();
        }
      })
      .onStepExit((response) => {
        // { element, index, direction }
      });

    // setup resize event
    window.addEventListener("resize", scroller.resize);
  });

  const padding = { top: 0, right: 15, bottom: 30, left: 15 };

  let width = null;
  let height = null;

  const grouped = mapToArray(d3.group(data, (d) => d.color_hex)).sort(
    (a, b) => a.value.length - b.value.length
  );

  for (let i = 0; i < grouped.length; i++) {
    // Lookup and append color name/text color to grouped data
    grouped[i].colors = data.find((d) => d.color_hex == grouped[i].key).colors;
    grouped[i].text_color = data.find(
      (d) => d.color_hex == grouped[i].key
    ).text_color;
  }

  const unique_colors = grouped.length;
  const maxColor = d3.max(grouped, (d) => d.value.length);

  $: xScaleBar = d3
    .scaleLinear()
    .domain([0, maxColor])
    .range([padding.left, width - padding.right]);

  $: yScaleBar = d3
    .scaleBand()
    .domain(grouped.map((d) => d.key))
    .range([height - padding.bottom, padding.top]);

  $: xTicks = xScaleBar.ticks(5);

  // FOR TIMELINE VIEW
  const num_paintings = 403;

  $: xScaleTimeline = d3
    .scaleLinear()
    .domain([0, num_paintings])
    .range([padding.left, width - padding.right]);

  $: yScaleTimeline = d3
    .scaleBand()
    .domain(grouped.map((d) => d.key))
    .range([height - padding.bottom, padding.top]);

  $: activeStep = 0;

  // SCROLL STEPS
  function init() {
    d3.selectAll(".timelineRect")
      .data(data)
      .transition()
      .duration(1000)
      .attr("x", 0);

    d3.selectAll(".colorBar")
      .data(grouped)
      .transition()
      .duration(1000)
      .delay(1000)
      .attr("width", (d) => xScaleBar(d.value.length))
      .attr("height", (height / unique_colors) * 0.9);
  }

  function createTimeline() {
    d3.selectAll(".colorBar")
      .data(grouped)
      .transition()
      .duration(1000)
      .attr("width", 0);

    d3.selectAll(".timelineRect")
      .data(data)
      .transition()
      .duration(1000)
      .delay(1000)
      .attr("x", (d) => xScaleTimeline(d.painting_index))
      .attr("opacity", 1);
  }

  function highlight() {
    d3.selectAll(".timelineRect")
      .data(data)
      .transition()
      .duration(1000)
      .attr("x", (d) => xScaleTimeline(d.painting_index))
      .attr("opacity", (d) =>
        (d.color_hex == "#8A3324") | (d.color_hex == "#5F2E1F") ? 1 : 0.3
      );
  }
</script>

<div class="scrollama-container">
  <div class="scrollama-graphic">
    <div class="chart" bind:offsetWidth={width} bind:offsetHeight={height}>
      <div class="timelineTip" />
      <svg style="width: 100%; height: 100%;">
        <ColorViz
          {width}
          {height}
          {padding}
          {grouped}
          {data}
          {xScaleBar}
          {yScaleBar}
          {xScaleTimeline}
          {yScaleTimeline}
          {xTicks}
        />
      </svg>
    </div>
  </div>

  <div class="scrollama-steps" id="colorSection">
    <div class="step" class:active={activeStep == 0} data-step="a">
      <p>Content.</p>
    </div>
    <div class="step" class:active={activeStep == 1} data-step="b">
      <p>Content.</p>
    </div>
    <div class="step" class:active={activeStep == 2} data-step="c">
      <p>
        Here, we notice patterns. At the end of Season 6, Bob Ross moved away
        from <span
          class="highlight-text"
          style="background: #8A3324; color: white;">Burnt Umber</span
        >
        and moved toward
        <span class="highlight-text" style="background: #5F2E1F; color: white;"
          >Dark Sienna</span
        >.
      </p>
      <p>
        The two colors are similar enough that Ross could easily substitute one
        for the other without much functional difference in his art.
      </p>
    </div>
  </div>
</div>

<style lang="scss">
  .timelineTip {
    opacity: 0;
    position: absolute;
    pointer-events: none;
  }

  :global(.timelineTip .title) {
    text-align: center;
    padding: 0.5rem;
    border-radius: 10px;
    box-shadow: 1px 1px 6px 1px rgba(0, 0, 0, 0.5);
  }
</style>