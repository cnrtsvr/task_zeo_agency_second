<template>
    <ag-grid-vue ref="agGrid" style="height: 800px"
                 class="ag-theme-balham"
                 :columnDefs="columnDefs"
                 :animateRows="true"
                 :gridOptions="gridOptions"
                 :rowSelection="rowSelection"
                 :toolPanel="toolPanel"
                 :defaultColDef="defaultColDefs"
                 :sideBar="true"
                 v-on:grid-ready="onGridReady"
                 :rowModelType="'infinite'"
                 :paginationPageSize="100"
                 :infiniteInitialRowCount="1">
    </ag-grid-vue>
</template>

<script>
    import {AgGridVue} from 'ag-grid-vue';
    import 'ag-grid-enterprise';
    import {LicenseManager} from "ag-grid-enterprise";

    LicenseManager.setLicenseKey("Evaluation_License-_Not_For_Production_Valid_Until_7_March_2019__MTU1MTkxNjgwMDAwMA==401412254de511491000d2684e818e25");
    export default {
        name: 'GridComponentInfiniteSCroll',
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
                    enableRowGroup: false,
                    enablePivot: false,
                    sortable: true,
                    filter: true
                },
                toolPanel: 'columns',
                rowSelection: 'multiple',
                gridOptions: {},
                gridApi: null,
                gridColumnApi: null,
                updateDataFn: null
            };
        },
        mounted() {
            this.gridApi = this.gridOptions.api;
            this.gridColumnApi = this.gridOptions.columnApi;
            this.$emit('init');
        },
        watch: {
            rowData: function () {
                this.updateDataFn(this.rowData);
            }
        },
        methods: {
            onGridReady(params) {
                this.updateDataFn = data => {
                    let dataSource = {
                        rowCount: null,
                        getRows: function(params) {
                            let dataAfterSortingAndFiltering = sortAndFilter(data, params.sortModel, params.filterModel);
                            let rowsThisPage = dataAfterSortingAndFiltering.slice(params.startRow, params.endRow);
                            let lastRow = -1;
                            if (dataAfterSortingAndFiltering.length <= params.endRow) {
                                lastRow = dataAfterSortingAndFiltering.length;
                            }
                            params.successCallback(rowsThisPage, lastRow);
                        }
                    };
                    params.api.setDatasource(dataSource);
                };
            }
        }
    }

    function sortAndFilter(allOfTheData, sortModel, filterModel) {
        return sortData(sortModel, filterData(filterModel, allOfTheData));
    }

    function sortData(sortModel, data) {
        let sortPresent = sortModel && sortModel.length > 0;
        if (!sortPresent) {
            return data;
        }
        let resultOfSort = data.slice();
        resultOfSort.sort(function(a, b) {
            for (let k = 0; k < sortModel.length; k++) {
                let sortColModel = sortModel[k];
                let valueA = a[sortColModel.colId];
                let valueB = b[sortColModel.colId];
                if (valueA === valueB) {
                    continue;
                }
                let sortDirection = sortColModel.sort === "asc" ? 1 : -1;
                if (valueA > valueB) {
                    return sortDirection;
                } else {
                    return sortDirection * -1;
                }
            }
            return 0;
        });
        return resultOfSort;
    }

    function filterData(filterModel, data) {
        let filterPresent = filterModel && Object.keys(filterModel).length > 0;
        if (!filterPresent) {
            return data;
        }
        let resultOfFilter = [];
        for (let i = 0; i < data.length; i++) {
            let item = data[i];
            if (filterModel.date) {
                if (filterModel.date.values.indexOf(item.date) < 0) {
                    continue;
                }
            }
            if (filterModel.keyword) {
                if (filterModel.keyword.values.indexOf(item.keyword) < 0) {
                    continue;
                }
            }
            if (filterModel.country) {
                if (filterModel.country.values.indexOf(item.country) < 0) {
                    continue;
                }
            }
            if (filterModel.device) {
                if (filterModel.device.values.indexOf(item.device) < 0) {
                    continue;
                }
            }
            if (filterModel.url) {
                if (filterModel.url.values.indexOf(item.url) < 0) {
                    continue;
                }
            }
            if (filterModel.position) {
                if (filterModel.position.values.indexOf(item.position) < 0) {
                    continue;
                }
            }
            resultOfFilter.push(item);
        }
        return resultOfFilter;
    }
</script>

<style lang="scss" scoped>
    @import "~ag-grid-community/dist/styles/ag-grid.css";
    @import "~ag-grid-community/dist/styles/ag-theme-balham.css";
</style>