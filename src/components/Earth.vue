<template>
  <div class="earth">
    <div class="tooltip-container"></div>
  </div>
</template>

<script>
import { geoMercator as mercator, geoPath as path, geoGraticule as graticule, geoCentroid as centroid } from 'd3-geo';
import { select, event as d3Event } from 'd3-selection';
import { interpolateOrRd } from 'd3-scale-chromatic';
import { scaleSequential } from 'd3-scale';
import { csv, json } from 'd3-fetch';
import * as topojson from 'topojson';

export default {
  name: 'Earth',
  components: {},
  data() {
    return {
      lifeSpan: 5000,
      fetchSpeed: 5000,
      covidCountryData: [],
      projection: '',
      path: '',
      graticule: '',
      svg: '',
      rowByName: [],
      maxNumber: 0,
      offsetL: 0,
      offsetT: 0,
      lastDate: '3/18/20'
    };
  },
  methods: {},
  async beforeCreate() {},
  async mounted() {
    const topoJSONdata = await json('https://cdn.jsdelivr.net/npm/world-atlas@2/countries-50m.json');

    let filePath =
      'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_19-covid-Confirmed.csv';
    this.covidCountryData = await csv(filePath);

    this.rowByName = this.covidCountryData.reduce((accumulator, d) => {
      let country = d['Country/Region'];
      let data = d;
      if (+d[this.lastDate] > this.maxNumber) {
        this.maxNumber = +d[this.lastDate];
      }
      if (country == 'US') {
        country = 'United States of America';
      }
      if (accumulator[country]) {
        let tmp = accumulator[country];
        for (const property in tmp) {
          data[property] = +data[property] + +tmp[property];
        }
      }
      accumulator[country] = data;
      return accumulator;
    }, {});

    const countries = topojson.feature(topoJSONdata, topoJSONdata.objects.countries);
    countries.features.forEach((d) => {
      Object.assign(d.properties, this.rowByName[d.properties.name]);
    });

    this.projection = mercator().precision(0.1);

    this.path = path().projection(this.projection);
    this.graticule = graticule();

    let earth = select(this.$el);
    this.offsetL = this.$el.offsetLeft + 10;
    this.offsetT = this.$el.offsetTop + 10;

    let colorScale = scaleSequential(interpolateOrRd).domain([0, this.maxNumber]);
    this.svg = earth
      .append('svg')
      .attr('preserveAspectRatio', 'xMidYMid meet')
      .attr('viewBox', '0 0 900 600')
      .attr('class', 'earth-responsive');

    this.svg
      .append('defs')
      .append('path')
      .datum({ type: 'Sphere' })
      .attr('id', 'sphere')
      .attr('d', this.path);

    this.svg
      .append('use')
      .attr('class', 'stroke')
      .attr('xlink:href', '#sphere');

    this.svg
      .append('path')
      .datum(this.graticule)
      .attr('class', 'graticule')
      .attr('d', this.path);

    const tooltip = select('.tooltip-container')
      .append('div')
      .attr('class', 'tooltip hidden');

    this.svg
      .selectAll('.country')
      .data(countries.features)
      .enter()
      .insert('path', '.graticule')
      .attr('class', 'country')
      .attr('d', this.path)
      .attr('fill', (d) => {
        if (d.properties[this.lastDate]) {
          return colorScale(+d.properties[this.lastDate]);
        } else {
          return '#E5ECF6';
        }
      })
      .on('mousemove', (d) => {
        const label = `${d.properties.name} ${typeof(d.properties[this.lastDate]) !== 'undefined' ? d.properties[this.lastDate] + ' confirmed' : ''}`;

        tooltip
          .classed('hidden', false)
          .attr('style', `left:${d3Event.pageX + this.offsetL}px;top:${d3Event.pageY + this.offsetT}px`)
          .html(label);
      })
      .on('mouseout', () => {
        tooltip.classed('hidden', true);
      });

    this.svg
      .insert('path', '.graticule')
      .datum(
        topojson.mesh(topoJSONdata, topoJSONdata.objects.countries, (a, b) => {
          return a !== b;
        })
      )
      .attr('class', 'boundary')
      .attr('d', this.path);

    this.svg // country names don't look good so don't show them
      .selectAll('text')
      .data(countries.features)
      .enter()
      .append('text')
      .text(function(d) {
        if (d.properties['Country/Region']) {
          return d.properties.name;
        } else {
          return '';
        }
      })
      .attr('class', 'country-name')
      .attr('x', (d) => {
        return this.projection(centroid(d))[0];
      })
      .attr('y', (d) => {
        return this.projection(centroid(d))[1];
      });
  },
};
</script>

<style>
.earth {
  display: inline-block;
  position: relative;
  width: 100%;
  padding-bottom: 100%; /* aspect ratio */
  vertical-align: top;
  overflow: hidden;
}
.earth-responsive {
  display: inline-block;
  position: absolute;
  top: 0;
  left: 0;
}
.stroke {
  fill: none;
  stroke: #000;
  stroke-width: 3px;
}
.fill {
  fill: #e5ecf6;
}
.graticule {
  fill: none;
  /* stroke: #777; */
  stroke: #777;
  stroke-width: 0.5px;
  stroke-opacity: 0.5;
}
.country {
  /*fill: #333344;*/
  /* fill: red; */
  stroke: #555566;
  stroke-width: 0.5px;
}
.boundary {
  fill: none;
  stroke: #555566;
  stroke-width: 0.5px;
}
.country-name {
  stroke: #333344;
  font-size: 10px;
  display: none;
}

div.tooltip {
  z-index: 2;
  color: #222;
  background: #fff;
  border-radius: 3px;
  box-shadow: 0px 0px 2px 0px #a6a6a6;
  padding: 0.2em;
  text-shadow: #f5f5f5 0 1px 0;
  opacity: 0.9;
  position: absolute;
}
</style>
