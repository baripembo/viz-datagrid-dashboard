<script>
  import * as d3 from 'd3';
  import Papa from 'papaparse'
  import { onMount } from 'svelte';
  import KeyFigure from './lib/KeyFigure.svelte'
  import Bar from './lib/Charts/Bar.svelte'
  import Map from './lib/Map.svelte'

  const randomData = (seriesCount) => {
    return d3.range(seriesCount).map(function (d) {
      return {
        date: d,
        value: Math.random(d)*100
      };
    })
  };

  let countryData = [];
  Papa.parse('https://hapi-testing.innovation.humdata.org/api/location?output_format=csv&limit=1000&offset=0', {
    header: true,
    download: true,
    complete: function(results) {
      countryData = results.data
    }
  })

  // const url = 'https://stage.hapi-humdata-org.ahconu.org/api/admin1?location_code=afg&output_format=json&offset=0';
  // let admin1Data = [];

  //dummy pie data
  let requirement = 90;
  let funded = 15;
  let fundedPercent = funded/requirement;
  let pieData = [1-fundedPercent, fundedPercent];

  //dummy key figures
  let keyFigureData = [
    {title: 'Key figure', value: '1,200,000', series: randomData(10)},
    {title: 'Key figure', value: '300000', series: randomData(10)},
    {title: 'Key figure', value: '1100000000', valueFormat:'$.2s'},
    {title: 'Key figure', value: '1100', series: randomData(5), seriesType: 'column'},
    {title: 'Key figure', value: '500'},
    {title: 'Key figure', value: '500', series: pieData, seriesType: 'pie'},
  ];

  let tabs = [
    {title: 'Population', id: 'population'},
    {title: '3W', id: '3w'},
    // {title: 'People in Need', id: ''},
    // {title: 'Internally Displaced People', id: ''},
    // {title: 'Refugees', id: ''},
    // {title: 'Returnees', id: ''},
    // {title: 'Organizations', id: ''},
    // {title: 'Food Insecurity', id: ''},
    // {title: 'Malnutrition', id: ''},
    // {title: 'Health', id: ''},
  ];

  let sidebarWidth, scrollingWrapperHeight, scrollingWrapper;

  let countrySelect = 'AFG'
  $: selected = countryData.find((c) => {
    return c.code === countrySelect 
  });

  //Total key figure
  $: totalValue = 0;

  $: rankingData = [];
  $: metadata = {};
  $: currentLayer = 'population';

  function getPopulationbyCountry(iso3) {
    Papa.parse(`https://hapi-testing.innovation.humdata.org/api/themes/population?location_code=${iso3}&admin_level=1&output_format=csv&limit=1000&offset=0`, {
      header: true,
      download: true,
      complete: function(results) {
        if (results.data[0] !== undefined) {
          let metadataURL = `https://hapi-testing.innovation.humdata.org/api/resource?hdx_id=${results.data[0].resource_hdx_id}&output_format=csv`;
          getMetadata(metadataURL);
        }

        let popByAdm = d3.rollup(results.data, v => d3.sum(v.filter(d => d.age_range_code === '' && d.gender_code === ''), d => d.population), d => d.admin1_name)
        let arr = [];
        popByAdm.forEach(function(value, key) {
          if (key !== undefined)
            arr.push({name: key, value: value});
        })

        rankingData = arr;
        totalValue = d3.sum(popByAdm.values());
      }
    })
  }

  function getOrgsbyCountry(iso3) {
    Papa.parse(`https://hapi-testing.innovation.humdata.org/api/themes/3W?location_code=${iso3}&output_format=csv&offset=0`, {
      header: true,
      download: true,
      complete: function(results) {
        if (results.data[0] !== undefined) {
          let metadataURL = `https://hapi-testing.innovation.humdata.org/api/resource?hdx_id=${results.data[0].resource_hdx_id}&output_format=csv`;
          getMetadata(metadataURL);
        }

        let orgsByAdm = d3.rollup(results.data, v => v.length, d => d.admin1_name);
        let arr = [];
        orgsByAdm.forEach(function(value, key) {
          if (key !== undefined)
            arr.push({name: key, value: value});
        })

        rankingData = arr;
        totalValue = d3.sum(orgsByAdm.values());
      }
    })
  }

  function getMetadata(url) {
    Papa.parse(url, {
      header: true,
      download: true,
      complete: function(results) {
        metadata.date = results.data[0].update_date;
        metadata.datasetURL = results.data[0].dataset_hdx_link;
        metadata.provider = results.data[0].dataset_hdx_provider_name;
      }
    })
  }

  function getCountryData(iso3) {
    if (currentLayer === '3w')
      getOrgsbyCountry(iso3);
    else
      getPopulationbyCountry(iso3);
  }

  function onLayerChange(id) {
    currentLayer = id;

    if (currentLayer === '3w')
      getOrgsbyCountry(countrySelect);
    else
      getPopulationbyCountry(countrySelect);
  }

  onMount(async() => {
    // const response = await fetch(url);
    // admin1Data = await response.json();
    // console.log(admin1Data)
    getPopulationbyCountry(countrySelect);

    //calculate available space for ranking chart
    scrollingWrapperHeight = window.innerHeight - scrollingWrapper.getBoundingClientRect().top - 80;
    scrollingWrapper.style.height = scrollingWrapperHeight + 'px';
  });
</script>

<main>
  <div class='select-wrapper'>
    <select bind:value={countrySelect} on:change={() => getCountryData(countrySelect)}>
      {#each countryData as {code, name}}
        {#if code!=''}
          <option value={code}>{name}</option>
        {/if}
      {/each}
    </select>
  </div> 


  {#if selected!=undefined}
    <h1>{selected.name}</h1>
  {/if}
  
  <h2 class='header'>Header</h2>
  <div class='grid-container key-figure-container'>
    {#each keyFigureData as keyFigure}
      <div class='col-2'>
        <KeyFigure {...keyFigure} />
      </div>
    {/each}
  </div>

  <h2 class='header'>Header</h2>

  <div class='tabs'>
    {#each tabs as {title, id}, i}
      <div class='tab' class:active={id === currentLayer} on:click={() => onLayerChange(id)}><a href='#' {title}>{title}</a></div>
    {/each}
  </div>

<!--   <div class='select-wrapper'>
    <select>
      {#each tabs as {title, id}}
        <option value={id}>{title}</option>
      {/each}
    </select>
  </div> -->

  <div class='content grid-container'>
    <div class='sidebar col-3' bind:clientWidth={sidebarWidth}>
      <KeyFigure title={'Total'} value={totalValue} metadata={metadata} /><!-- series={randomData(10)} -->

      <h3 class='chart-title'>Ranking</h3>
      <div class='scrolling-wrapper' bind:this={scrollingWrapper}>
        {#if sidebarWidth>0}
          <div class='ranking-container'>
            <Bar data={rankingData} width={sidebarWidth} />
          </div>
        {/if}
      </div>

    </div>
    <div class='main-content col-9'>
      <Map center={[67, 34]} zoom={5} />
    </div>
  </div>
</main>

<style lang='scss'>
  .key-figure-container {
    padding-left: 15px;
  }
  .main-content  {
    margin: 0;
  }
  .sidebar {
    margin-right: 0;
    padding-top: 15px;
  }
  .chart-title {
    margin-top: 15px;
  }
  .scrolling-wrapper {
    overflow-y: scroll;
  }
</style>
