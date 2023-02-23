<template>
  <div class="list-meeting" >
    <div class="title">Meetings</div>
    <div class='vuetable-wrapper' id="list-items" :class="{ 'loading' : isLoading }">
      <img class="loader loading" style="margin-top:20%" src="/icons/loader.svg" v-if="isLoading">
      <vuetable
          ref="vuetable"
          :field="fieldTable"
          :api-mode="false"
          api-url=""
          :data="meetings"
          pagination-path="" class="table table-striped service-container"
          :append-params="filterParams"
          :no-data-template="$t('No data found')"
          @vuetable:row-clicked="clickMeeting"
          @vuetable:loading="isLoading = true"
          @vuetable:loaded="isLoading = false">
        <template slot="elmec-filter" slot-scope="props" @click.stop.prevent="() => {}">
          <TableFiltersMaker :key="refresh" @tableFiltersRefresh="tableFiltersRefresh" :header="props.field"
                             @changedDateFrom="changedDateFrom" @changedDateTo="changedDateTo" :filters="tableFilters"/>
        </template>
        <template slot="type" slot-scope="props">
         <div v-if="props.rowData.mainMeeting.type !== null">{{props.rowData.mainMeeting.type}}</div>
          <div v-else>{{' - '}}</div>
        </template>
        <template slot="category" slot-scope="props" >
          <div v-for="category in meetingsCategory" :key="category.value">
            <div v-if="props.rowData.mainMeeting.title === category.value">
              {{category.value}}
           </div>
          </div>
        </template>
        <template slot="dataIncontro" slot-scope="props">
          <div>{{ formatDate(props.rowData.dataIncontro) }}</div>
        </template>
        <template slot="division" slot-scope="props" >
          <div style="width: 50px; margin-left: 15%" v-if="props.rowData.mainMeeting.idDivision === 2 ">
            <img :src="'/icons/security/cybergonC.svg'" height="20" v-tooltip.right="{ content: $t('managedByCyb') }"/>
          </div>
          <div v-else style="margin-left: 15%">
            <icon scale="1.3"
                  :name="'my-elmec-logo-colored' "
                  v-tooltip.right="{ content: $t('managedByElm') }"></icon>
          </div>
        </template>
<!--        <template slot="pinned" slot-scope="props" >-->
<!--          <div v-if="props.rowData.pinned === true">-->
<!--            <img  :src="'/icons/pinned.svg'" height="20"/>-->
<!--          </div>-->
<!--        </template>-->
      </vuetable>
    </div>
  </div>
</template>

<script>
import {workflowMixins} from '@/mixins/workflow-mixins'
import TableFiltersMaker from '@/components/global/filters/TableFiltersMaker'
import {Vuetable} from 'ElmecTable'
let moment = require('moment')
export default {
  components: { TableFiltersMaker, Vuetable },
  name: 'meeting-table',
  mixins: [workflowMixins],
  computed: {
    fieldTable: function () {
      return [
        {
          title: '__slot:elmec-filter',
          name: 'title'
        },
        {
          title: '__slot:elmec-filter',
          name: '__slot:dataIncontro',
        },
        {
          title: '__slot:elmec-filter',
          name: '__slot:type',
        },
        {
          title: '__slot:elmec-filter',
          name: '__slot:category',
        },
        {
          title: '__slot:elmec-filter',
          name: 'status'
        },
        {
          title: '__slot:elmec-filter',
          name: 'mode'
        },
        {
          title: '__slot:elmec-filter',
          name: '__slot:division',
        },
        // {
        //   title: '',
        //   name: '__slot:pinned',
        // }
      ]
    },
    tableFilters () {
      return [
        {
          filter_name: 'title',
          type: 'text',
          label: 'Meeting',
          field: 'title',
          width: '80%'
        },
        {
          filter_name: '__slot:dataIncontro',
          type: 'datepicker',
          range: true,
          showCancel: true,
          label: 'Data',
          field: 'dataIncontroDa',
          fieldTo: 'dataIncontroA'
        },
        {
          filter_name: '__slot:type',
          label: this.$t('type'),
          displayValue: 'key',
          valueName: 'value',
          options:this.filtroPerTipo,
          field: 'type',
          type: 'select',
        },
        {
          filter_name: '__slot:category',
          label: this.$t('category'),
          displayValue: 'key',
          valueName: 'value',
          options: this.filtroCategoria,
          field: 'category',
          type: 'select',
        },
        {
          filter_name: 'status',
          field: 'status',
          type: 'select',
          label: this.$t('status'),
          displayValue: 'displayValue',
          valueName: 'value',
          value: '',
          options: [
            {value: 'completed', displayValue: 'Completed'},
            {value: 'planned', displayValue: 'Planned'}
          ]
        },
        {
          filter_name: 'mode',
          field: 'mode',
          type: 'select',
          label: this.$t('mode'),
          displayValue: 'displayValue',
          valueName: 'value',
          value: '',
          options: [
            {value: 'live', displayValue: 'Live'},
            {value: 'remote', displayValue: 'Remote'}
          ]
        },
        {
          filter_name: '__slot:division',
          field: 'division',
          type: 'select',
          label:this.$t('managedBy'),
          displayValue: 'displayValue',
          valueName: 'value',
          imgValue: 'img',
          value: '',
          options: [
            {value: '2', displayValue: 'Cyber Security Manager',  img: '/icons/security/cybergonC.svg'},
            {value: '1', displayValue: 'CSM',  img:'/icons/elmecSoloLogo.svg'}
          ]
        }
      ]
    }
  },
  data () {
    return {
      refresh: 0,
      filterParams: {},
      dataTableItems: [],
      dataFormat: '',
      taskSelectedID: '',
      valueSearch: '',
      categoryValue: '',
      isLoading: false,
      filtroPerTipo: [],
      filtroCategoria: [],
      typeSelected: '',
      typeOptions: [],
      notFilteredMeetings: [],
      taskSelected: {},
      typeValue: '',
      typeKey: '',
      categorySelected: '',
      meetingsCategory: [],
      defaultFilters: [],
      meetingsTemp: [],   //Utilizzato per salvare i main meeting
      meetings: [],
      myHeight: window.innerHeight,
      serviceIconMapping: {
        infrastructure: '/icons/your-services/infrastructure.svg',
        workplace: '/icons/your-services/workplace.svg',
        security: '/icons/menumodules/colored/soc.svg'
      }
    }
  },
  mounted: function () {
        this.loadMeeting()
  },
  watch: {
  },
  methods: {
    // RowClass(row) {
    //   return row.pinned === true ? 'trpinned' : ''
    // },
    formatDate(row) {
     return moment(row).format('DD-MM-YYYY')
    },
    loadMeeting () {
      // this.isLoading = true
      this.$chttp.tamtasks.get(process.env.VUE_APP_WS_TOOLS + '/api/mye/tam/customer/' + this.getUser().codiCli + '/meetings', {headers: {'X-Source': 'MYTSK'}}).then((resp) => {
        this.isLoading = true
        if (resp && resp.status === 200) {
          let data = resp.body
          if (data && data.length) {
            let staticRequest = []
            data.forEach((elem) => {
              this.meetingsTemp.push(elem)
              if (!this.typeOptions.includes(elem.item.type)) {
                this.typeOptions.push(elem.item.type)
              }
              if (!this.meetingsCategory.includes(elem.item.title)) {
                this.meetingsCategory.push(elem.item.title)
              }
              staticRequest.push(this.$chttp.tamtasks.get(process.env.VUE_APP_WS_TOOLS + '/api/mye/tam/customer/' + this.getUser().codiCli + '/meeting/' + elem.item.id + '/submeetings', {headers: {'X-Source': 'MYTSK'}}))
            })
            Promise.all(staticRequest.map(function (route) {
              return route.catch(function (err) {
                return err
              })
            })).then((responses) => {
              responses.forEach((resp, index) => {
                if (resp && resp.body && resp.body.length) {
                  resp.body.forEach((elem) => {
                    let obj = elem.item
                    obj.mainMeeting = this.meetingsTemp[index].item
                    this.meetings.push(obj)
                  })
                }
              })
              if (this.meetings.length) {
                this.typeOptions.forEach((option, index) => {
                  this.typeOptions[index] = {value: option ? option : 'null', key: option ? option : 'Not Defined'}
                  this.filtroPerTipo = this.typeOptions
                })
                this.meetingsCategory.forEach((option, index) => {
                  this.meetingsCategory[index] = {value: option ? option : 'null', key: option ? option : 'Not Defined'}
                  this.filtroCategoria = this.meetingsCategory
                })
                this.meetings.sort((a, b) => {
                  let keyA = (a.dataIncontro)
                  let keyB = (b.dataIncontro)
                  if (keyA < keyB) return 1
                  if (keyA > keyB) return -1
                  return 0
                })
                if (this.$route.params.idTask) {
                  this.meetings.forEach((element, index) => {
                    if (Number(this.$route.params.idTask) === Number(element.id)) {
                      this.taskSelectedID = element.id
                      this.clickMeeting(this.meetings[index])
                    }
                  })
                }
                this.isLoading = false
              }
            })
          }
          if (data.length === 0 ){
            this.isLoading = false
          }
        }
      })
    },
    clickMeeting: function (meeting) {
      this.taskSelected = meeting
      let sharingObj = meeting
      sharingObj.type = meeting.mainMeeting.type
      this.taskSelectedID = sharingObj.id
      this.$emit('click-task', sharingObj, this.meetingsCategory)
      let name = this.hasGovernanceMenu() ? 'TasksSrvMngmParams' : 'TasksSrvMngmParams-manage'
      this.$router.push({
        name: name,
        params: {
          idTask: meeting.id
        }})
    },
    tableFiltersRefresh: function (filters) {
      this.filterParams = {
        ...this.defaultFilters,
        ...this.$route.query,
        ...filters
      }
      let listElements = this.notFilteredMeetings
      if (this.filterParams.title && this.filterParams.title !== '') {
        listElements = listElements.filter((d) => {
          return d.title.toLowerCase().includes(this.filterParams.title.toLowerCase())
        })
      }
      if (this.filterParams.type) {
        listElements = listElements.filter((d) => {
          if( d.mainMeeting.type !== null) {
            return d.mainMeeting.type.toLowerCase().includes(this.filterParams.type.toLowerCase())
          }
        })
      }
      if (this.filterParams.dataIncontroDa || this.filterParams.dataIncontroA) {
        listElements = listElements.filter((d) => {
          if( d.mainMeeting.dataIncontro !== null) {
            let dataMoment = moment(d.dataIncontro).format('YYYY-MM-DD')
            if (this.filterParams.dataIncontroDa && this.filterParams.dataIncontroA) {
              if (this.filterParams.dataIncontroDa < dataMoment && this.filterParams.dataIncontroA > dataMoment) {
                return d
              }
            } else if (this.filterParams.dataIncontroDa) {
              if (this.filterParams.dataIncontroDa < dataMoment) {
                return d
              }
            } else if (this.filterParams.dataIncontroA) {
              if (this.filterParams.dataIncontroA > dataMoment) {
                return d
              }
            }
          }
        })
      }
      if (this.filterParams.category) {
        listElements = listElements.filter((d) => {
          return d.mainMeeting.title.toLowerCase().includes(this.filterParams.category.toLowerCase())
        })
      }
      if (this.filterParams.status && this.filterParams.status !== '') {
        listElements = listElements.filter((d) => {
          return d.status.toLowerCase().includes(this.filterParams.status.toLowerCase())
        })
      }
      if (this.filterParams.mode && this.filterParams.mode !== '') {
        listElements = listElements.filter((d) => {
          return d.mode.toLowerCase().includes(this.filterParams.mode.toLowerCase())
        })
      }
      if (this.filterParams.division && this.filterParams.division !== '') {
        listElements = listElements.filter((d) => {
          return d.mainMeeting.idDivision.toString().includes(this.filterParams.division)
        })
      }
      this.meetings = listElements
      this.$forceUpdate()
      if (this.$refs.vuetable) {
        this.$nextTick(function () {
          this.$refs.vuetable.refresh()
        })
      }
    },
    changedDateFrom: function (dateFrom) {
      this.$router.replace({query: {...this.$route.query, ...{dataIncontroDa: dateFrom}}})
      this.tableFiltersRefresh()
    },
    changedDateTo: function (dateTo) {
      this.$router.replace({query: {...this.$route.query, ...{dataIncontroA: dateTo}}})
      this.tableFiltersRefresh()
    }
  },
  i18n: {
    messages: {
      it: {
        mode: 'Modalità',
        status: 'Stato',
        managedBy: 'Owner',
        managedByElm: 'CSM',
        managedByCyb:'Cyber Security Manager',
        category: 'Categoria',
        type: 'Tipologia',
        state: 'Stato'
      },
      en: {
        mode: 'Mode',
        status: 'Status',
        managedBy: 'Owner',
        managedByElm: 'CSM',
        managedByCyb:'Cyber Security Manager',
        category: 'Category',
        type: 'Type',
        state: 'Status'
      }
    }
  }
}
</script>

<style lang="less">
@import "../../style/less/font";
@import "../../style/less/variabili";
.list-meeting {
  margin-top: 10px;
  overflow-y: auto;

  .title {
    margin-bottom: 30px;
    margin-top: 10px;
    .set-font-bold(@size: 30px; @color: @grayBorder);
  }

  #list-items {
    min-height: 500px;
    height: 100%;

    .service-header {
      padding: 10px 20px;

      .text-header {
        text-transform: capitalize;
        .set-font-bold(@size: 14px; @color: @labelGray);
        text-align: left;
      }

     border-bottom: 1px solid @myLightGray-100;
    }

    .service-row {
      display: flex;
      align-items: center;
      text-transform: capitalize;

      .service-icon {
        margin-right: 5px;
      }
    }

    .service-container {
      tbody {
        //.trpinned {
        //  background-color: lightgoldenrodyellow!important;
        //}
        tr {
       background-color: @myWhite;
          height: 50px;
          width: 100%;
          cursor: pointer;

          .label-row {
            text-transform: capitalize;
          }
        }

        tr:hover {
          background-color: @lightGray !important;
        }
      }
    }

  }
}
</style>
