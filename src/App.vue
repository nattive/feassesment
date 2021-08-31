<template>
  <div class="body">
    <td class="seach-container">
      <p>Search</p>
      <div class="search-box">
        <input
          v-model="searchInput"
          @change="onSearch"
          class="search"
          type="text"
        />
        <a href="#" @click="clearSearchInput" v-if="searchInput">X</a>
      </div>
    </td>
    <div class="table">
      <table>
        <tr>
          <th>Name</th>
          <th>ISBN</th>
          <th>Authors</th>
          <th>Pages</th>
          <th>Country</th>
          <th>Released</th>
        </tr>
        <tr v-if="this.error">
          <td colspan="6" style="color: red">
            {{ this.error }} <a href="#" @click="getBooks">Refresh</a>
          </td>
        </tr>
        <tr v-else v-for="(book, index) in booksArray" :key="index">
          <td>{{ book.name }}</td>
          <td>{{ book.isbn }}</td>
          <td>{{ book.authors }}</td>
          <td>{{ book.numberOfPages }}</td>
          <td>{{ book.country }}</td>
          <td>{{ book.released }}</td>
        </tr>
      </table>

      <div class="footer">
        <div class="details">
          <p>
            Showing 1 to {{ booksArray.length }} of
            {{ booksArray.length }} entries
          </p>
        </div>
        <div class="pagination">
          <a
            disabled
            href="#"
            @click.prevent="gotoPage(parginate === 1 ? 1 : parginate - 1)"
            >Previous</a
          >
          <a
            href="#"
            :class="activePage === parginate ? 'active' : null"
            @click.prevent="gotoPage(parginate)"
            v-for="parginate in 2"
            :key="parginate"
            >{{ parginate }}</a
          >
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import "./assets/styles.css";
import axios from "axios";
export default {
  name: "App",
  data() {
    return {
      pageSize: 10,
      page: 1,
      activePage: 1,
      isLoading: false,
      books: [],
      searchedResult: [],
      searchInput: undefined,
      error: undefined,
    };
  },
  computed: {
    booksArray: function() {
      if (this.searchInput) {
        return this.searchedResult;
      } else {
        return this.books;
      }
    },
  },
  methods: {
    GetFormattedDate(pdate) {
      var date = new Date(pdate),
        month = "" + (date.getMonth() + 1),
        day = "" + date.getDate(),
        year = date.getFullYear();

      if (month.length < 2) month = "0" + month;
      if (day.length < 2) day = "0" + day;

      return [year, month, day].join("-");
    },
    clearSearchInput() {
      this.searchInput = undefined;
      this.searchedResult = [];
    },
    gotoPage(number) {
      this.page = number;
      this.activePage = number;
      this.getBooks();
    },
    next() {
      this.page = this.pageSize++;
      this.getBooks();
    },
    onSearch() {
      //  const regest = RegExp("[\w\W]*")
      //  https://www.anapioficeandfire.com/api/books?page=${this.page}&pageSize=${this.pageSize}

      const result = this.books.filter((book) => {
        if (book.name.includes(this.searchInput)) {
          return true;
        }
      });
      this.searchedResult = [];
      console.log(result);
      this.searchedResult = result;
    },
    getBooks() {
      const vm = this;
      vm.isLoading = true;
      vm.error = undefined;
      axios
        .get(
          `https://www.anapioficeandfire.com/api/books?page=${this.page}&pageSize=${this.pageSize}`
        )
        .then((response) => {
          vm.books = [];
          response.data?.map((book) =>
            vm.books.push({
              name: book.name,
              isbn: book.isbn,
              authors: book.authors.toString(),
              numberOfPages: book.numberOfPages,
              country: book.country,
              released: this.GetFormattedDate(book.released),
            })
          );
        })
        .catch(() => (this.error = "An error occurred"));
    },
  },
  mounted() {
    this.getBooks();
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
