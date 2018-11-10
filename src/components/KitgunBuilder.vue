<template>
  <div class="kitgunbuilder">
    <el-steps :active="part" finish-status="success">
      <el-step :title="$t('kitgun.selectChamber')"></el-step>
      <el-step :title="$t('kitgun.selectGrip')"></el-step>
      <el-step :title="$t('kitgun.selectLoader')"></el-step>
    </el-steps>
    <!-- 枪膛 -->
    <ul class="partlist" v-if="part === 0">
      <li v-for="item in chamberList" :key="item.id">
        <el-radio class="part" v-model="chamber" :label="item" border>
          <div class="snapshot">
            <img :src="`/img/kitgunChamber${item.name.replace(/ /g, '')}.png`" :alt="item.name" width="100%">
          </div>
          <div class="name">
            {{$t(`messages.${item.name}`)}}
          </div>
          <div class="type">
            {{$t(`kitgun.type.${item.name}`)}}
          </div>
        </el-radio>
      </li>
    </ul>
    <!-- 握把 -->
    <ul class="partlist" v-if="part === 1">
      <li v-for="item in gripList" :key="item.id">
        <el-radio class="part" v-model="grip" :label="item" border>
          <div class="snapshot">
            <img :src="`/img/kitgunGrip${item.name.replace(/ /g, '')}.png`" :alt="item.name" width="100%">
          </div>
          <div class="name">
            {{$t(`messages.${item.name}`)}}
          </div>
          <div class="prop">
            <span>{{loadGrip(item).fireRate}} {{$t(`kitgun.fireRate`)}}</span>
            <span>{{loadGrip(item).dmgAdd >= 0 ? "+" : ""}}{{loadGrip(item).dmgAdd}} {{$t(`kitgun.damage`)}}</span>
          </div>
        </el-radio>
      </li>
    </ul>
    <!-- 填弹器 -->
    <ul class="partlist" v-if="part === 2">
      <li v-for="item in loaderList" :key="item.id">
        <el-radio class="part" v-model="loader" :label="item" border>
          <div class="snapshot">
            <img :src="`/img/kitgunLoader${item.name.replace(/ /g, '')}.png`" :alt="item.name" width="100%">
          </div>
          <div class="name">
            {{$t(`messages.${item.name}`)}}
          </div>
          <div class="prop">
            <span>{{loadLoader(item).critDamage+2}}x {{$t(`kitgun.critDamage`)}}</span>
            <span>{{loadLoader(item).critChance >= 0 ? "+" : ""}}{{(loadLoader(item).critChance*100).toFixed()}}% {{$t(`kitgun.critChance`)}}</span>
            <span>{{loadLoader(item).procChance >= 0 ? "+" : ""}}{{(loadLoader(item).procChance*100).toFixed()}}% {{$t(`kitgun.status`)}}</span>
            <span>{{loadLoader(item).reload >= 0 ? "+" : ""}}{{loadLoader(item).reload}}s {{$t(`kitgun.reload`)}}</span>
            <span>{{loadLoader(item).magazine >= 0 ? "+" : ""}}{{loadLoader(item).magazine}} {{$t(`kitgun.magazine`)}}</span>
          </div>
        </el-radio>
      </li>
    </ul>
    <!-- 部件 -->
    <div class="parts">
      <div class="part" v-if="chamber">{{$t("kitgun.chamber")}}: {{$t(`messages.${chamber.name}`)}}</div><!--
   --><div class="part" v-if="grip">{{$t("kitgun.grip")}}: {{$t(`messages.${grip.name}`)}}</div><!--
   --><div class="part" v-if="loader">{{$t("kitgun.loader")}}: {{$t(`messages.${loader.name}`)}}</div>
    </div>
    <!-- 预览 -->
    <div class="preview" v-if="chamber">
      <div class="prop">{{$t("kitgun.damage")}}: {{+kitgun.panelDamage.toFixed(1)}}</div><!--
   --><div class="prop" v-for="dmg in kitgun.dmg" :key="dmg[0]">{{$t(`elements.${dmg[0]}`)}}: {{dmg[1]}}</div><!--
   --><div class="prop">{{$t("kitgun.fireRate")}}: {{kitgun.fireRate}}</div><!--
   --><div class="prop">{{$t("kitgun.status")}}: {{(kitgun.status*100).toFixed()}}%</div><!--
   --><div class="prop">{{$t("kitgun.critDamage")}}: {{kitgun.critMul}}x</div><!--
   --><div class="prop">{{$t("kitgun.critChance")}}: {{(kitgun.critChance*100).toFixed()}}%</div><!--
   --><div class="prop">{{$t("kitgun.magazine")}}: {{kitgun.magazine}}</div><!--
   --><div class="prop">{{$t("kitgun.reload")}}: {{kitgun.reload}}</div>
    </div>
    <el-button class="stepctl" :disabled="part === 0" @click="part = part > 0 ? part - 1 : 0">{{$t("kitgun.lastStep")}}</el-button>
    <el-button class="stepctl" :disabled="part === 0 && !chamber || part === 1 && !grip || part === 2 && !loader" @click="part = part < 2 ? part + 1 : (finish(),2)">{{part === 2 ? $t("kitgun.finish") : $t("kitgun.nextStep")}}</el-button>
  </div>
</template>
<script lang="ts">
import _ from "lodash";
import { Vue, Component, Watch, Prop } from "vue-property-decorator";
import { Kitgun, KitgunChamberData, KitgunGripData, KitgunLoaderData, KitgunChamber, KitgunGrip, KitgunLoader } from "@/warframe";

@Component
export default class extends Vue {
  part = 0;
  chamberList = KitgunChamberData;
  gripList = KitgunGripData;
  loaderList = KitgunLoaderData;
  chamber: KitgunChamber = null;
  grip: KitgunGrip = null;
  loader: KitgunLoader = null;

  get kitgun() { return new Kitgun(this.chamber, this.grip, this.loader); }

  loadGrip(grip: KitgunGrip) { return Kitgun.loadGrip(this.chamber, grip); }
  loadLoader(loader: KitgunLoader) { return Kitgun.loadLoader(this.chamber, loader); }

  finish() {
    this.$emit("finish", this.kitgun);
  }

}

</script>

<style lang="less">
.preview .prop,
.parts .part {
  display: inline-block;
  margin: 8px 4px 4px;
  padding: 4px 8px;
  border: 1px solid #6199ff;
  border-radius: 4px;
  color: #6199ff;
  font-size: 0.9em;
}
.kitgunbuilder {
  .el-steps.el-steps--horizontal {
    margin: 8px 16px;
  }
  .stepctl {
    margin-top: 12px;
  }
  .partlist {
    text-align: center;
    display: flex;
    flex-wrap: wrap;
    // justify-content: space-between;
    li {
      margin: 4px 2px;
    }
    .part {
      padding: 8px 16px;
    }
    .el-radio {
      height: auto;
    }
    .el-radio__label {
      padding: 0;
    }
    .snapshot {
      width: 110px;
      height: 80px;
    }
    .name {
      font-size: 1.1em;
    }
    .type {
      margin-top: 8px;
      color: #aaa;
    }
    .prop {
      margin-top: 8px;
      span {
        display: block;
        margin-top: 4px;
      }
    }
    .is-checked .type {
      color: #9cbfff;
    }
  }
}
</style>