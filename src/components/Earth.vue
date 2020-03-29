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
import { json } from 'd3-fetch';
import * as topojson from 'topojson';
import countryMismatch from '@/utils/countries';

export default {
  name: 'Earth',
  components: {},
  data() {
    return {
      topoJSONdata: null,
      covidCountryData: [],
      projection: '',
      path: '',
      graticule: '',
      svg: '',
      countries: null,
      maxNumber: 0,
      confirmed: 'totalCases',
      test: null
    };
  },
  methods: {
    SetCountryData: async function() {
      this.topoJSONdata = await this.getTopoJSONData()
      const rowByName = this.covidCountryData.reduce((accumulator, d) => {
        let country = d['name'];
        let data = d;
        if (country != 'Total' && +d[this.confirmed] > this.maxNumber) {
          this.maxNumber = +d[this.confirmed];
        }
        if (countryMismatch[country]) {
          country = countryMismatch[country];
        }
        accumulator[country.replace(/ /g,"")] = data;
        return accumulator;
      }, {});
      this.countries = topojson.feature(this.topoJSONdata, this.topoJSONdata.objects.countries);
      this.countries.features.forEach((d) => {
          Object.assign(d.properties, rowByName[d.properties.name.replace(/ /g,"")]);
      });
    },
    getTopoJSONData: async function() {
      return json('https://cdn.jsdelivr.net/npm/world-atlas@2/countries-50m.json')
    }
  },
  async beforeCreate() {},
  async mounted() {

    let filePath =
      'https://exchange.vcoud.com/coronavirus/latest';
    this.covidCountryData = await json(filePath);

    await this.SetCountryData();

    this.projection = mercator().precision(0.1);

    this.path = path().projection(this.projection);
    this.graticule = graticule();

    let earth = select(this.$el);

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
      .data(this.countries.features)
      .enter()
      .insert('path', '.graticule')
      .attr('class', 'country')
      .attr('d', this.path)
      .attr('fill', (d) => {
        if (d.properties[this.confirmed]) {
          return colorScale(+d.properties[this.confirmed]);
        } else {
          return '#E5ECF6';
        }
      })
      .on('mousemove', (d) => {
        const label = `${d.properties.name} ${
          typeof d.properties[this.confirmed] !== 'undefined' ? d.properties[this.confirmed] + ' confirmed' : ''
        }`;

        tooltip
          .classed('hidden', false)
          .attr('style', `left:${d3Event.pageX + 10}px;top:${d3Event.pageY + 10}px`)
          .html(label);
      })
      .on('mouseout', () => {
        tooltip.classed('hidden', true);
      });

    this.svg
      .insert('path', '.graticule')
      .datum(
        topojson.mesh(this.topoJSONdata, this.topoJSONdata.objects.countries, (a, b) => {
          return a !== b;
        })
      )
      .attr('class', 'boundary')
      .attr('d', this.path);

    this.svg // country names don't look good so don't show them
      .selectAll('text')
      .data(this.countries.features)
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
