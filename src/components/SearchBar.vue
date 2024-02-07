<!-- src/components/BookSearch.vue -->

<template>
  <div>
    <nav class="navbar navbar-expand-lg navbar-light bg-light">
      <div class="container">
        <button
          class="navbar-toggler"
          type="button"
          data-toggle="collapse"
          data-target="#navbarNav"
          aria-controls="navbarNav"
          aria-expanded="false"
          aria-label="Toggle navigation"
          >
        <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNav">
          <form class="form-inline" @submit.prevent="searchBooks">
            <div class="row">
              <div class="col">
                <input
                  v-model="keyword"
                  @focus="isSearchFocused = true"
                  @blur="isSearchFocused = false"
                  class="form-control"
                  type="search"
                  placeholder="Search for books..."
                  />
              </div>
              <div class="col-auto">
                <button class="btn btn-outline-success" type="button">Search</button>
              </div>
            </div>
          </form>
        </div>
      </div>
    </nav>
    <!-- Display the dropdown only if there are search results -->
    <div v-if="books.length > 0 && searchFocused" class="container mt-3">
      <div class="dropdown">
        <button
          class="btn btn-secondary dropdown-toggle"
          type="button"
          id="dropdownMenuButton"
          data-toggle="dropdown"
          aria-haspopup="true"
          aria-expanded="false"
          >
        Search Results
        </button>
        <div class="dropdown-menu" aria-labelledby="dropdownMenuButton">
          <a
            v-for="book in books"
            :key="book.id"
            @click="addToLog(book)"
            class="dropdown-item"
            href="#"
            >
          {{ book.volumeInfo.title }} by {{ book.volumeInfo.authors && book.volumeInfo.authors.join(', ') }}
          </a>
        </div>
      </div>
    </div>
    <div class="container mt-3">
      <!-- Display the books in the container as before -->
      <div v-for="book in books" :key="book.id" class="card mb-3">
        <div class="card-body">
          <h5 class="card-title">{{ book.volumeInfo.title }}</h5>
          <p class="card-text">by {{ book.volumeInfo.authors && book.volumeInfo.authors.join(', ') }}</p>
          <button @click="addToLog(book)" class="btn btn-primary">Add to log</button>
        </div>
      </div>
    </div>
  </div>
  Reading log:
  <div class="log-container" v-if="log.length > 0">
    <div v-for="b in log" :key="b.id" class="log-book">
      <img v-if="b.volumeInfo.imageLinks" :src="b.volumeInfo.imageLinks.thumbnail" :alt="b.volumeInfo.title">
      <br/>
      <span>{{ b.volumeInfo.title }}</span>
      <template v-if="b.saleInfo.retailPrice && b.saleInfo.currencyCode == 'USD'">
        ({{ b.saleInfo.retailPrice.amount.toLocaleString('en-US', { style: 'currency', currency: 'USD' }) }})
      </template>
      <template v-else-if="b.saleInfo.listPrice && b.saleInfo.currencyCode == 'USD'">
        ({{ b.saleInfo.listPrice.amount.toLocaleString('en-US', { style: 'currency', currency: 'USD' }) }})
      </template>
    </div>
  </div>
  <br/>
  Savings: {{ savings.toLocaleString('en-US', { style: 'currency', currency: 'USD' }) }}
  <br/>
  Books Read: {{booksRead}}
</template>


<style>
.log-container {
  display: flex;
  flex-wrap: wrap;
}

.log-book {
  width: 30%; /* Adjust the width based on your layout */
  margin: 10px;
  padding: 10px;
  border: 1px solid #ccc;
  box-sizing: border-box;
}

.log-book img {
  max-width: 100%;
  height: auto;
  margin-bottom: 10px;
}


.book-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between; /* or use 'space-around' based on your preference */
}
.input-group-append {
 padding-left: 15px;
}

.book-item {
  width: 30%; /* Adjust the width based on your layout */
  margin-bottom: 20px; /* Add margin to create space between items */
  padding: 10px;
  border: 1px solid #ccc;
  box-sizing: border-box;
}
.search-bar {
  display: flex;
  align-items: center;
  background-color: #fff;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.search-bar input {
  flex: 1;
  padding: 8px;
  border: none;
  outline: none;
  font-size: 14px;
}


</style>

<script>
export default {
  data() {
    return {
      books: [],
      keyword: '',
      orderBy: 'newest',
      maxResults: '10',
      loadState: '',
      log: [],
      showSearch: true,
      savings: 0,
      booksRead: 0,
      isSearchFocused: false,
    };
  },

//   fetch(`https://www.googleapis.com/books/v1/volumes?q=search-terms&key=your-API-key)
//   .then(response => response.json())
//   .then(result => {
// this.setState({ books: result.items})
// })}
  methods: {
    async searchBooks() {
      //this.searchFocused = true;
      this.$axios
        .get(
          `https://www.googleapis.com/books/v1/volumes?q=intitle:${
            this.keyword
          }&orderBy=${this.orderBy}&maxResults=${this.maxResults}`
        )
        .then(response => {
          console.log(response.data.items)
          this.books = response.data.items
          console.log(this.books)
          this.loadState = 'success'
        })
    },
    addToLog(book) {
      this.log.push(book);
      this.showSearch = false;
      this.books = [];
      this.keyword = '';
      this.booksRead++;
      //this.searchFocused = false;
      if (book.saleInfo && book.saleInfo.retailPrice) {
        this.savings += book.saleInfo.retailPrice.amount;
      } else if (book.saleInfo && book.saleInfo.listPrice) {
        this.savings += book.saleInfo.listPrice.amount;
      }
    },
    getBookCoverUrl(coverId) {
      return `http://covers.openlibrary.org/b/id/${coverId}-L.jpg`;
    },
    addPrice(amount) {
      this.savings += amount;
    }
  },
  watch: {
    keyword() {
      // Call searchBooks whenever the keyword changes (user types)
      if (this.isSearchFocused) this.searchBooks();
    },
  },
};
</script>
