<template>
    <ag-grid-vue ref="agGrid" style="height: 800px"
                 class="ag-theme-balham"
                 :columnDefs="columnDefs"
                 :animateRows="true"
                 :gridOptions="gridOptions"
                 :rowSelection="rowSelection"
                 :defaultColDef="defaultColDefs"
                 :sideBar="sideBar"
                 v-on:grid-ready="onGridReady"
                 :rowModelType="'serverSide'"
                 :paginationPageSize="100"
                 :infiniteInitialRowCount="1">
    </ag-grid-vue>
</template>

<script>
    import {AgGridVue} from 'ag-grid-vue';
    import 'ag-grid-enterprise';
    import {LicenseManager} from "ag-grid-enterprise";
    import * as _ from "lodash";

    LicenseManager.setLicenseKey("Evaluation_License-_Not_For_Production_Valid_Until_7_March_2019__MTU1MTkxNjgwMDAwMA==401412254de511491000d2684e818e25");
    export default {
        name: 'GridComponentServerSide',
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
                    enableValue: false,
                    enableRowGroup: true,
                    enablePivot: false,
                    sortable: true,
                    filter: "agSetColumnFilter",
                },
                sideBar: {
                    toolPanels: [
                        {
                            id: "columns",
                            labelDefault: "Columns",
                            labelKey: "columns",
                            iconKey: "columns",
                            toolPanel: "agColumnsToolPanel",
                            toolPanelParams: {
                                suppressRowGroups: false,
                                suppressValues: true,
                                suppressPivots: true,
                                suppressPivotMode: true,
                                suppressSideButtons: true,
                                suppressColumnFilter: false,
                                suppressColumnSelectAll: false,
                                suppressColumnExpandAll: false
                            }
                        }
                    ],
                    defaultToolPanel: "columns"
                },
                rowSelection: 'multiple',
                gridOptions: {},
                gridApi: null,
                gridColumnApi: null
            };
        },
        mounted() {
            this.gridApi = this.gridOptions.api;
            this.gridColumnApi = this.gridOptions.columnApi;
            this.$emit('init');
        },
        watch: {
          rowData: function () {
              this.onGridReady(null);
          }
        },
        methods: {
            onGridReady(params) {
                let datasource = createServerSideDatasource(this.rowData);
                if(params) {
                    params.api.setServerSideDatasource(datasource);
                } else {
                    this.gridApi.setServerSideDatasource(datasource);
                }
            }
        }
    }

    function createServerSideDatasource(rowData) {
        function IServerSideDatasource(rowData) {
            this.rowCount = null;
            this.getRows = function (params) {
                //console.log("ServerSideDatasource.getRows: params = ", params);
                let request = params.request;
                let dataCopy = rowData ? [...rowData] : [];
                let groupKeys = request.groupKeys;
                let rowGroupCols = request.rowGroupCols;
                let filterModel = request.filterModel;
                let sortModel = request.sortModel;
                let gonnaGroup = rowGroupCols.length > 0;
                let processData = [];
                if (gonnaGroup) {
                    let rowGroupColIds = rowGroupCols.map(x => x.id);
                    let parentId = groupKeys.length > 0 ? groupKeys.join("") : "";
                    processData = group(dataCopy, rowGroupColIds, groupKeys, parentId);
                    processData = sortAndFilter(processData, sortModel, filterModel);
                } else {
                    processData = sortAndFilter(dataCopy, sortModel, filterModel);
                }

                let resultForGrid = getBlockFromResult(processData, request);
                let lastRow = getLastRowResult(processData, request);

                params.successCallback(resultForGrid, lastRow);
            };
        }

        return new IServerSideDatasource(rowData);
    }

    function getBlockFromResult (data, request) {
        return data.slice(request.startRow, request.endRow);
    }

    function getLastRowResult (result, request) {
        let lastRowFound = result.length <= request.endRow;
        return lastRowFound ? result.length : null;
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
        resultOfSort.sort(function (a, b) {
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

    function group(data, rowGroupColIds, groupKeys, parentId) {
        let groupColId = rowGroupColIds.shift();
        if (!groupColId) return data;
        let groupedData = _.groupBy(data, groupColId);
        if (groupKeys.length === 0) {
            return Object.keys(groupedData).map(key => {
                let res = {};
                res["id"] = getGroupId(parentId, key);
                res[groupColId] = key;
                return res;
            });
        }
        return group(groupedData[groupKeys.shift()], rowGroupColIds, groupKeys, parentId);
    }

    function getGroupId(parentId, key) {
        return parentId ? parentId + "-" + key : key;
    }
</script>

<style lang="scss">
    @import './gridStyles.scss';
</style>

