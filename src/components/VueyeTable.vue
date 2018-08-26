<template>
<div class="vet-container" :style="theme.main">
    <div class="vet-header">
        <div class="vet-header-title">
            <h4> {{title}}</h4>
        </div>
        <div class="vet-header-search-container">
            <input type="text" name="search" placeholder="Search ... " class="vet-header-search"
             v-model="searchValue" :style="theme.borderBottom">
            
            <i class="icon search-icon" style="margin-left:-18px"></i>
        </div>
        <div class="themes">
          <span>Theme: </span>
          <div class="light" @click="changeTheme('light')"></div>
          <div class="dark-1" @click="changeTheme('dark-1')"></div>
          <div class="dark-2" @click="changeTheme('dark-2')"></div>

        </div>
        <div class="vet-header-btns">
            <i class="icon document-icon" @click="exportTableToExcel"></i>
            <i class="icon print-icon" @click="printTable()"></i>
        </div>

    </div>
    <table :id="title|lowercase" >
        <thead>
          <th  :style="getTableStyle"></th>
            <th  :style="getTableStyle" v-for="(col,index) in columns" :key="col">
                <div class="tab-head-cell">
                    <span>{{col}}</span>
                    <span class="tab-head-cell-icons">
                 <i class="icon x-small-icon up-icon" @click="sort('asc',index)"></i>                    
                 <i class="icon x-small-icon down-icon"  @click="sort('desc',index)"></i>
               </span>
                </div>
            </th>
        </thead>
        <tbody>
            <tr class="data-row" :style="{color:currentTheme.main.color,backgroundColor:currentTheme.main.backgroundColor}" v-for="(row,indx) in currentPageData" :id="indx"  @click="clickOnRow(row,indx)">
               <td  :style="getTableStyle">       <label class="container">
  <input type="checkbox" checked="checked">
  <span class="checkmark" :id="indx"></span>
</label></td>
                <td  :style="getTableStyle" class="tab-rows_data-cell" v-for="(cell,key,index)  in row" :key="key+index"> {{cell}}</td>
            </tr>
        </tbody>
    </table>

    <div class="vet-footer">
        <div class="vet-footer-perpage">
            <span>Number of rows per page </span>
            <select v-model="nbRowPerPage" @change="recreatePages">
                     <option v-for="ppval in perPageValues " :value="ppval" :key="ppval">{{ppval}}</option>
                     
            </select>
        </div>
        <div class="vet-footer-page-desc">
            <span> {{lowerBound}} <strong>-</strong> {{upperBound}} of {{rows_data.length}} </span>
        </div>
        <div class="vet-footer-pagination">
            <i class="icon small-icon start-icon" @click="gotoFirstPage"></i>
            <i class="icon small-icon back-icon" @click="gotoPrevPage"></i>
            <i class="icon small-icon forward-icon" @click="gotoNextPage"></i>
            <i class="icon small-icon end-icon" @click="gotoLastPage"></i>

        </div>

    </div>

</div>
</template>

<script>
export default {
  name: "vueye-table",
  /**************************************** */
  /**************************************** */
  /**************************************** */
  /****************@prop *************** */
  /**************************************** */
  /**************************************** */
  props: {
    title: {
      type: String,
      default: "my-table"
    },
    cols: {
      type: Array,
      default: () => []
    },
    rows_data: {
      type: Array,
      default: () => []
    },
    perPageValues: {
      type: Array,
      default: () => [5, 10, 25, 50, 100, 500]
    },
    themeName: {
      type: String
    },
    tableStyle: {
      type: String,
      default: "striped"
    }
  },
  /**************************************** */
  /**************************************** */
  /**************************************** */
  /****************@data *************** */
  /**************************************** */
  /**************************************** */
  data() {
    return {
      searchValue: "",
      currentPageIndex: 0,
      nbRowPerPage: 10,
      nbPages: 0,
      paginated_data: {},
      currentTheme: {
        main: {},
        borderBottom: {},
        rowClick: {}
      }
    };
  },
  filters: {
    lowercase(val) {
      return val
        .toString()
        .toLowerCase()
        .replace(/ /g, "-");
    }
  },
  /**************************************** */
  /**************************************** */
  /**************************************** */
  /****************@computed ************** */
  /**************************************** */
  /**************************************** */
  computed: {
    columns() {
      if (this.cols.length == 0) {
        if (typeof this.rows_data[0] == "object") {
          return Object.keys(this.rows_data[0]);
        }
      } else {
        return this.cols;
      }
    },
    /**@function currentPageData locates to the first page by default
     * or to the page we are going to by paginating
     */
    currentPageData() {
      if (this.searchValue == "") {
        this.createPages();
        return this.paginated_data[this.currentPageIndex];
      } else {
        this.createPagesAfterSearch();
        return this.paginated_data[this.currentPageIndex];
      }
    },
    /**@function lowerBound in the current page show lower bound of showed data compared to the origin data */
    lowerBound() {
      return this.currentPageIndex * this.nbRowPerPage + 1;
    },
    /**@function upperBound in the current page show upper bound of showed data compared to the origin data */
    upperBound() {
      return (this.currentPageIndex + 1) * this.nbRowPerPage >
        this.rows_data.length
        ? this.rows_data.length
        : (this.currentPageIndex + 1) * this.nbRowPerPage;
    },
    theme() {
      return this.themeName != undefined || this.themeName != ""
        ? this.changeTheme(this.themeName)
        : this.changeTheme("light");
    },
    getTableStyle() {
      if (this.tableStyle == "striped") {
        return {
          borderBottom: this.currentTheme.borderBottom.borderBottom
        };
      } else {
        return { border: this.currentTheme.borderBottom.borderBottom };
      }
    }
  },
  /**************************************** */
  /**************************************** */
  /**************************************** */
  /****************@methods *************** */
  /**************************************** */
  /**************************************** */

  methods: {
    /*** @method createPages  creates pages with default per page value */
    createPages() {
      /** @description lengthAll for getting the length of the array containing all data */
      let lengthAll = Object.keys(this.rows_data).length;
      this.nbPages = 0;
      /** @description We slice our initial data according to the @nbRowPerPage */
      for (let i = 0; i < lengthAll; i = i + this.nbRowPerPage) {
        this.paginated_data[this.nbPages] = this.rows_data.slice(
          i,
          i + this.nbRowPerPage
        );
        this.nbPages++;
      }
    },
    createPagesAfterSearch() {
      this.currentPageIndex = 0;
      /** @description lengthSearch for getting the length of the array containing searched data */
      let lengthSearch = Object.keys(this.search()).length;

      if (lengthSearch !== 0) {
        this.nbPages = 0;
        /** @description We slice our searched data according to the @nbRowPerPage */
        for (let i = 0; i < lengthSearch; i = i + this.nbRowPerPage) {
          this.paginated_data[this.nbPages] = this.search().slice(
            i,
            i + this.nbRowPerPage
          );
          this.nbPages++;
        }
      } else {
        this.paginated_data = [];
      }
    },

    /** @method recreatePages  creates pages when we select a new per page value
     *  and set the index to the first page
     */
    recreatePages() {
      this.currentPageIndex = 0;
      this.createPages();
    },
    gotoFirstPage() {
      this.currentPageIndex = 0;
    },
    gotoPrevPage() {
      this.currentPageIndex > 0
        ? this.currentPageIndex--
        : (this.currentPageIndex = 0);
    },
    gotoNextPage() {
      this.currentPageIndex < this.nbPages - 1
        ? this.currentPageIndex++
        : (this.currentPageIndex = this.nbPages - 1);
    },
    gotoLastPage() {
      this.currentPageIndex = this.nbPages - 1;
    },
    compareValues(key, order) {
      return function(a, b) {
        if (!a.hasOwnProperty(key) || !b.hasOwnProperty(key)) {
          // property doesn't exist on either object
          return 0;
        }

        let varA = typeof a[key] === "string" ? a[key].toUpperCase() : a[key];
        let varB = typeof b[key] === "string" ? b[key].toUpperCase() : b[key];

        let comparison = 0;
        if (varA > varB) {
          comparison = 1;
        } else if (varA < varB) {
          comparison = -1;
        }
        return order === "desc" ? comparison * -1 : comparison;
      };
    },
    /**@method sort this method sort our data by specifiying the sort type and the column to be sorted */
    sort(sortDirection, index) {
      let tmp_data = this.rows_data.sort(
        this.compareValues(Object.keys(this.rows_data[1])[index], sortDirection)
      );
    },
    /**@method search return the items which values start with searched value
     * you could specify another criteria to search by modifying this method
     */
    search() {
      if (this.searchValue != "") {
        let searchedData = [];
        searchedData = this.rows_data.filter(item => {
          let values = Object.values(item).filter(val => {
            return val
              .toString()
              .toUpperCase()
              .startsWith(this.searchValue.toUpperCase());
          });
          if (values.length != 0) {
            return item;
          }
        });
        return searchedData;
      }
    },
    clickOnRow(row, index) {
      let els = document.getElementsByClassName("data-row");
      for (let i = 0; i < els.length; i++) {
        els[i].style.backgroundColor = "inherit";
      }
      let e = document.getElementById("" + index);
      this.currentTheme.main.rowClick != undefined
        ? (e.style.backgroundColor = this.currentTheme.main.rowClick.backgroundColor)
        : (e.style.backgroundColor = "#fafafa");
      this.$emit("click-row", row);
    },
    exportTableToExcel() {
      let downloadLink;
      let dataType = "application/vnd.ms-excel";
      let tableSelect = this.createTable();
      let tableHTML = tableSelect.replace(/ /g, "%20");

      // Specify file name
      let filename = this.title + ".xls";

      // Create download link element
      downloadLink = document.createElement("a");

      document.body.appendChild(downloadLink);

      if (navigator.msSaveOrOpenBlob) {
        let blob = new Blob(["\ufeff", tableHTML], {
          type: dataType
        });
        navigator.msSaveOrOpenBlob(blob, filename);
      } else {
        // Create a link to the file
        downloadLink.href = "data:" + dataType + ", " + tableHTML;

        // Setting the file name
        downloadLink.download = filename;

        //triggering the function
        downloadLink.click();
      }
    },
    createTable() {
      let table = "<table><thead>";

      table += "<tr>";
      for (let i = 0; i < this.columns.length; i++) {
        const column = this.columns[i];
        table += "<th>";
        table += column;
        table += "</th>";
      }
      table += "</tr>";

      table += "</thead><tbody>";

      for (let i = 0; i < this.rows_data.length; i++) {
        const row = this.rows_data[i];
        table += "<tr>";
        for (let j = 0; j < Object.keys(this.rows_data[i]).length; j++) {
          table += "<td>";
          table += Object.values(this.rows_data[i])[j];
          table += "</td>";
        }
        table += "</tr>";
      }

      return (table += "</tbody></table>");
    },
    printTable: function() {
      let table = this.createTable();
      /******************** */
      let yourDOCTYPE = "<!DOCTYPE html..."; // your doctype declaration
      let printPreview = window.open("", "print_preview");
      let printDocument = printPreview.document;
      printDocument.open();
      let head =
        "<head>" +
        "<title>" +
        this.title +
        "</title>" +
        "<style>*{font-family:serif}table{border-collapse: collapse;width: 100%;} thead{background:#ddd;} td, th {border: 1px solid #000;text-align: center;padding: 10px;}</style>" +
        "</head>";

      printDocument.write(
        yourDOCTYPE +
          "<html>" +
          head +
          "<body><h2>" +
          this.title +
          " :</h2>" +
          table +
          "</body>" +
          "</html>"
      );
      printPreview.print();
      printPreview.close();

      /******************** */
    },
    changeTheme(theme) {
      let icons = document.getElementsByClassName("icon");

      if (theme == "dark-1") {
        this.currentTheme.main = {
          backgroundColor: "#201d31",
          color: "#00ffff",
          rowClick: {
            backgroundColor: "#201d44"
          }
        };
        this.currentTheme.borderBottom = {
          borderBottom: "1px solid #00ffff"
        };
        for (let i = 0; i < icons.length; i++) {
          icons[i].style.filter = "sepia(100%) hue-rotate(160deg) saturate(35)";
        }
      } else if (theme == "dark-2") {
        this.currentTheme.main = {
          backgroundColor: "#201d31",
          color: "#ffff00",
          rowClick: {
            backgroundColor: "#201d44"
          }
        };
        this.currentTheme.borderBottom = {
          borderBottom: "1px solid #ffff00"
        };
        for (let i = 0; i < icons.length; i++) {
          icons[i].style.filter = "sepia(100%) hue-rotate(15deg) saturate(100)";
        }
      } else {
        this.currentTheme.main = {
          backgroundColor: "#fff",
          color: "#6a6a6a",
          rowClick: {
            backgroundColor: "#fafafa"
          }
        };
        this.currentTheme.borderBottom = {
          borderBottom: "1px solid #6a6a6a"
        };
        for (let i = 0; i < icons.length; i++) {
          icons[i].style.filter = "none";
        }
      }

      return this.currentTheme;
    }
  },
  mounted() {}
};
</script>
<style src="./icons.css" lang="scss" scoped>
</style>

<style src="./vueye-table.scss" lang="scss" scoped>
</style>