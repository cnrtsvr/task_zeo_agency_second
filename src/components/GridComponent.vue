<template>
    <ag-grid-vue ref="agGrid" style="height: 800px"
                 class="ag-theme-balham"
                 :columnDefs="columnDefs"
                 :rowData="rowData"
                :animateRows="true"
                 :gridOptions="gridOptions"
                 :rowSelection="rowSelection"
                 :toolPanel="toolPanel"
                :defaultColDef="defaultColDefs"
                :sideBar="true"
                :pagination="true"
                :paginationAutoPageSize="true">
    </ag-grid-vue>
</template>

<script>
    import {AgGridVue} from 'ag-grid-vue';
    import 'ag-grid-enterprise';
    import {LicenseManager} from "ag-grid-enterprise";

    LicenseManager.setLicenseKey("Evaluation_License-_Not_For_Production_Valid_Until_7_March_2019__MTU1MTkxNjgwMDAwMA==401412254de511491000d2684e818e25");
    export default {
        name: 'GridComponent',
        components: {
            AgGridVue
        },
        props: {
            rowData: Array,
            columnDefs: Array
        },
        data() {
            return {
                defaultColDefs: {
                    enableValue: true,
                    enableRowGroup: true,
                    enablePivot: true,
                    sortable: true,
                    filter: true
                },
                toolPanel: 'columns',
                rowSelection: 'multiple',
                gridOptions: {},
                gridApi: null,
                gridColumnApi: null,
            };
        },
        mounted() {
            this.gridApi = this.gridOptions.api;
            this.gridColumnApi = this.gridOptions.columnApi;
            this.$emit('init');
        }
    }
</script>

<style lang="scss" scoped>
    @import "~ag-grid-community/dist/styles/ag-grid.css";
    @import "~ag-grid-community/dist/styles/ag-theme-balham.css";
</style>