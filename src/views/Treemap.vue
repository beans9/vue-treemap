<template>
  <div class="treemap">
    <!--    The SVG structure is explicitly defined in the template with attributes-->
    <!--    derived from component data-->
    <svg :height="height" style="margin-left: 0px;" :width="width">
      <g style="shape-rendering: crispEdges;" transform="translate(0,0)">
        <!-- We can use Vue transitions too! -->
        <transition-group name="list" tag="g" class="depth">
          <g class="children" v-for="children in selectedNode._children" :key="'c_' + children.id">
            <rect
              class="parent"
              :id="children.id"
              :key="children.id"
              :x="x(children.x0)"
              :y="y(children.y0)"
              :width="x(children.x1 - children.x0 + children.parent.x0)"
              :height="y(children.y1 - children.y0 + children.parent.y0)"
              :style="{ fill: '#ededed' }"
            >
              <!-- The title attribute -->
              <title>{{ children.data.name }} | {{ children.value }}</title>
              <text :x="x(children.x0)" :y="y(children.y0)">test</text>
            </rect>

            <text
              dy="0em"
              :key="'t_' + children.data.name"
              :id="'t_' + children.data.name"
              :x="x(children.x0 + x(children.x1 - children.x0 + children.parent.x0) / 2)"
              :y="y(children.y0 + y(children.y1 - children.y0 + children.parent.y0) / 2)"
              dominant-baseline="middle"
              text-anchor="middle"
              style="fill-opacity: 1;font-weight: bold;font-size: 1rem"
              :style="{ 'font-size': rankingFontSize(children.data) }"
            >
              {{ children.data.name }}
              <tspan
                style="fill-opacity: 1;font-weight: bold;font-size: 0.6rem;fill:red"
                :style="{ fill: rankingStyle(children.data) }"
              >
                {{ rankingLabel(children.data) }}
              </tspan>
            </text>

            <text
              dy="1.7em"
              :key="'t_' + children.id"
              :x="x(children.x0 + x(children.x1 - children.x0 + children.parent.x0) / 2)"
              :y="y(children.y0 + y(children.y1 - children.y0 + children.parent.y0) / 2)"
              dominant-baseline="middle"
              text-anchor="middle"
              style="font-size: 0.8rem;fill-opacity: 1;fill:#b7afae"
            >
              {{ countLabel(children.value) }}
            </text>
          </g>
        </transition-group>
      </g>
    </svg>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import { Component, Prop, Watch } from "vue-property-decorator";
import { scaleLinear, scaleOrdinal } from "d3-scale";
import { json } from "d3-request";
import { hierarchy, treemap } from "d3-hierarchy";
import { select, selectAll } from "d3-selection";

interface TreeMapModel {
  children: TreeMapData[];
}

interface TreeMapData {
  name: string;
  value: number;
  newRank: boolean;
  rankDiff: number;
  rank: number;
}

interface RootNode {
  x: number;
  y: number;
  x0: number;
  y0: number;
  x1: number;
  y1: number;
  depth: number;
}

type KeywordRankSpan = "1d" | "1w" | "1M";

export class KeywordRankList {
  date = "";

  span: KeywordRankSpan = "1w";

  chartItemList: KeywordRankItem[] = [];
}

export class KeywordRankItem {
  rank = 0;

  rankDiff = 0;

  keyword = "";

  frequency = 0;

  newRank = false;
}

export class KeywordRankResponse {
  trendKeywordRankChartList: KeywordRankList[] = [];
  brandKeywordRankChartList: KeywordRankList[] = [];
}

const d3 = {
  selectAll,
  scaleLinear,
  scaleOrdinal,
  json,
  hierarchy,
  treemap,
  select
};

@Component
export default class Treemap extends Vue {
  private data!: KeywordRankList;
  private jsonData: TreeMapModel | null = null;

  private rootNode: RootNode = {
    x: 0,
    y: 0,
    x0: 0,
    y0: 0,
    x1: 0,
    y1: 0,
    depth: 0
  };

  private margin = {
    top: 20,
    right: 0,
    bottom: 0,
    left: 0
  };

  private width = 1070;

  private height = 607;

  private dataLength = 0;

  mounted() {
    this.initialize();
  }

  private initialize() {
    this.data = {
      date: "2020-08-30",
      span: "1w",
      chartItemList: [
        {
          rank: 1,
          rankDiff: 5,
          keyword: "향좋은",
          frequency: 664,
          newRank: false
        },
        {
          rank: 2,
          rankDiff: 9,
          keyword: "보습",
          frequency: 278,
          newRank: false
        },
        {
          rank: 3,
          rankDiff: 1,
          keyword: "여름",
          frequency: 77,
          newRank: false
        },
        {
          rank: 4,
          rankDiff: -1,
          keyword: "대용량",
          frequency: 66,
          newRank: false
        },
        {
          rank: 5,
          rankDiff: 4,
          keyword: "퍼퓸",
          frequency: 64,
          newRank: false
        },
        {
          rank: 6,
          rankDiff: -5,
          keyword: "바이오가",
          frequency: 46,
          newRank: false
        },
        {
          rank: 7,
          rankDiff: 1,
          keyword: "좋은",
          frequency: 43,
          newRank: false
        },
        {
          rank: 8,
          rankDiff: -1,
          keyword: "향기좋은",
          frequency: 42,
          newRank: false
        },
        {
          rank: 9,
          rankDiff: 7,
          keyword: "남자",
          frequency: 37,
          newRank: false
        },
        {
          rank: 10,
          rankDiff: 7,
          keyword: "더바디샵화이트머스크",
          frequency: 32,
          newRank: false
        },
        {
          rank: 11,
          rankDiff: 0,
          keyword: "아토피",
          frequency: 32,
          newRank: true
        },
        {
          rank: 12,
          rankDiff: 6,
          keyword: "향기",
          frequency: 31,
          newRank: false
        },
        {
          rank: 13,
          rankDiff: -3,
          keyword: "등드름",
          frequency: 30,
          newRank: false
        },
        {
          rank: 14,
          rankDiff: -2,
          keyword: "명품",
          frequency: 30,
          newRank: false
        },
        {
          rank: 15,
          rankDiff: -2,
          keyword: "연예인",
          frequency: 26,
          newRank: false
        },
        {
          rank: 16,
          rankDiff: -14,
          keyword: "아이유",
          frequency: 20,
          newRank: false
        },
        {
          rank: 17,
          rankDiff: 2,
          keyword: "겨울",
          frequency: 8,
          newRank: false
        },
        {
          rank: 18,
          rankDiff: 2,
          keyword: "선물",
          frequency: 7,
          newRank: false
        },
        {
          rank: 19,
          rankDiff: 2,
          keyword: "순한",
          frequency: 5,
          newRank: false
        },
        {
          rank: 20,
          rankDiff: 2,
          keyword: "아기",
          frequency: 4,
          newRank: false
        },
        {
          rank: 21,
          rankDiff: 0,
          keyword: "여드름",
          frequency: 1,
          newRank: true
        },
        {
          rank: 22,
          rankDiff: 0,
          keyword: "임산부",
          frequency: 1,
          newRank: true
        }
      ]
    };

    this.jsonData = {
      children: []
    };
    const childrenData: any = [];
    this.data.chartItemList.forEach((d: KeywordRankItem) => {
      childrenData.push({
        name: d.keyword,
        value: d.frequency,
        newRank: d.newRank,
        rankDiff: d.rankDiff,
        rank: d.rank
      });
    });
    this.jsonData.children = childrenData;

    this.rootNode = hierarchy(this.jsonData)
      .eachBefore((d: any) => {
        d.id = (d.parent ? `${d.parent.id}.` : "") + d.data.name;
      })
      .sum((d: any) => d.value)
      .sort((a: any, b: any) => b.height - a.height || b.value - a.value);

    this.rootNode.x1 = this.width;
    this.rootNode.y1 = this.height;

    this.accumulate(this.rootNode, this);
    this.treemap(this.rootNode);
  }

  private accumulate(d: any, context: this) {
    d._children = d.children;
    if (d._children) {
      d.value = d._children.reduce((p: any, v: any) => p + context.accumulate(v, context), 0);
      return d.value;
    }
    return d.value;
  }

  private rankingFontSize(value: TreeMapData) {
    if (value.rank === 1) {
      return "1.7rem";
    }
    if (value.rank < 6) {
      return "1.3rem";
    }
    if (value.rank < 10) {
      return "1.1rem";
    }
    if (value.rank < 15) {
      return "0.9rem";
    }
    return "0.7rem";
  }

  private rankingLabel(data: TreeMapData) {
    if (data.newRank) {
      return "NEW";
    }
    const count = data.rankDiff;
    if (count === 0) {
      return "ㅡ";
    }
    if (count > 0) {
      return `▲${count}`;
    }
    return `▼${count * -1}`;
  }

  private rankingStyle(data: TreeMapData) {
    if (data.newRank) {
      return "#654eff";
    }
    const value = data.rankDiff;
    if (value === 0) {
      return "#cacfd4";
    }
    if (value > 0) {
      return "#ff2119";
    }
    return "#0071f6";
  }

  private countLabel(value: number) {
    return `${value.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",")}건`;
  }

  get x() {
    return d3
      .scaleLinear()
      .domain([0, this.width])
      .range([0, this.width]);
  }

  get y() {
    return d3
      .scaleLinear()
      .domain([0, this.height - this.margin.top - this.margin.bottom])
      .range([0, this.height - this.margin.top - this.margin.bottom]);
  }

  get treemap() {
    return d3
      .treemap()
      .size([this.width, this.height])
      .round(false)
      .paddingInner(0);
  }

  get selectedNode() {
    const node = this.rootNode;
    // Recalculates the y and x domains
    this.x.domain([node.x0, node.x0 + (node.x1 - node.x0)]);
    this.y.domain([node.y0, node.y0 + (node.y1 - node.y0)]);
    console.log("node,,", node);
    return node;
  }

  @Watch("data")
  private changeData() {
    this.initialize();
  }
}
</script>

<style scoped>
text {
  pointer-events: none;
}

.grandparent text {
  font-weight: bold;
}

rect {
  fill: none;
  stroke: #fff;
}

rect.parent,
.grandparent rect {
  stroke-width: 2px;
}

.grandparent rect {
  fill: orange;
}

.grandparent:hover rect {
  fill: #ee9700;
}

.children rect.parent,
.grandparent rect {
  cursor: pointer;
}

.children rect.parent {
  fill: #bbb;
  fill-opacity: 0.5;
}

.children:hover rect.child {
  fill: #bbb;
}

.children text {
  font-size: 0.8em;
}

.grandparent text {
  font-size: 0.9em;
}

.list-enter-active,
.list-leave-active {
  transition: all 1s;
}
.list-enter, .list-leave-to /* .list-leave-active for <2.1.8 */ {
  opacity: 0;
}
</style>
