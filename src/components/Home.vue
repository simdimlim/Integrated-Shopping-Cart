<template>
  <div class="home">
    <navbar />
    <div class="columns">
      <div class="column">
        <aside class="menu is-hidden-touch" style="text-align:left;margin-left:25px;margin-top:29px;font-weight:100">
          <p class="menu-label" style="margin-left:1px;font-weight:700">Search</p>
          <ul class="menu-list" style="padding-bottom:15px">
            <div class="field search-bar-field">
              <p class="control has-icons-left">
                <input class="input search-bar" v-model="searchText" style="box-shadow:none;height:30px;font-size:13px" placeholder="Search items">
                <span style="padding-top:6px" class="icon is-small is-left">
                  <i class="fa fa-search"></i>
                </span>
              </p>
            </div>
          </ul>
          <p class="menu-label" style="font-weight:700">Filter</p>
          <ul class="menu-list" style="font-size:13px;padding-bottom:10px">
            <li class="filter-radio">
              <input type="radio" name="answer" v-model="itemType" value="unfulfilled">
                Unpurchased Items
              </input>
            </li>
            <li class="filter-radio">
              <input type="radio" name="answer" v-model="itemType" value="fulfilled">
                Purchased Items
              </input>
            </li>
            <li>
              <input type="radio" name="answer" v-model="itemType" value="all">
                All
              </input>
            </li>
          </ul>
          <ul class="menu-list">
            <li><a style="padding-left:0px;font-size:13px">By Price</a></li>
            <div class="columns" style="margin-bottom:0px">
              <div class="column" style="padding-right:5px">
                <input class="input filter-price" type="number" v-model="minPrice" placeholder="Min">
              </div>
              <div class="column" style="padding-left:0">
                <input class="input filter-price" type="number" v-model="maxPrice" placeholder="Max">
              </div>
            </div>
            <li>
              <p style="padding-left:0px;font-size:13px;padding-top:10px;padding-bottom:8px">By Category</p>
              <div class="select is-primary" style="height:30px;font-size:13px;width:100%">
                <select v-model="category" style="border: solid 1px #00d3d1;width:100%">
                   <option disabled value="">Select a category</option>
                  <option v-for="c in categories">{{ c }}</option>
                </select>
              </div>
            </li>
            <span class="checkbox" style="padding-left:0px;font-size:13px;padding-top:20px;padding-bottom:20px" >
              <input type="checkbox" v-model="showFaves">
                Favourites Only
              </input>
            </span>
            <a class="button is-primary" v-on:click="resetFilters" style="height:30px;font-size:13px;">Reset Filters</a>
            <br>
          </ul>
          <p class="menu-label" style="font-weight:700">My Stores</p>
          <ul class="menu-list" style="font-size:13px;height:150px;overflow:auto">
            <li v-for="store in myStores" style="padding-bottom:10px">
              <input type="checkbox" v-model="selectedStores" v-bind:value="store" style="padding-left: 0px">
               {{ store }}
            </li>
          </ul>
        </aside>
      </div>
      <div class="column is-10" style="padding-left:30px;padding-right:30px">
        <section class="section" style="padding-top:20px;padding-left:0;padding-bottom:15px">
          <div class="container" style="padding-top:100px;width:auto;padding-right:268px" v-if="currentUser.name == ''">
             <div  style="padding-left:25%;">
                 <three-dots></three-dots>
              </div>
          </div>
          <div class="container" style="width:auto;" v-if="currentUser.name != ''">
            <div class="select is-pulled-right" style="z-index: 3;">
              <select v-model="sort" style="font-weight:100">
                <option>Popularity</option>
                <option>First added</option>
                <option>Last added</option>
                <option>Price low to high</option>
                <option>Price high to low</option>
              </select>
            </div>
            <p class="is-pulled-right" style="padding-right: 1%; padding-top: 0.5%;font-weight:100">Sort by: </p>
            <h1 class="title num-items">
              {{ filteredItems.length }} items from {{ numStoresFilteredList }} retailers
            </h1>
          </div>
        </section>
        <section class="section" style="padding-left:0;padding-top:0;" v-if="currentUser.name != ''">
          <section class="section" v-if="filteredItems.length == 0" style="text-align:left;padding-left:0;padding-top:10px;color:darkgrey">No items.</section>
            <paginate name="items" :list="filteredItems" :per="8" class="columns is-multiline">
              <div v-for="item in filteredItems" v-model="currentUser.items" class="column is-3">
                <itemcard :title="item.title" :price="item.price" :img="item.img" :category="item.category" :timestamp="item.timestamp" :host="item.host" :link="item.link" :favourite="item.favourite" :itemId="item.key"
                  :purchased="item.purchased"></itemcard>
              </div>
            </paginate>
            <div class="columns">
              <div class="column" style="text-align:left">
            <button class="button" v-if="currentPage != 0" @click="firstPage"><<</button>
            <button class="button" v-if="currentPage != 0" @click="prevPage">Prev</button>
            </div>
            <div class="column" style="text-align:center;font-weight:100">Page {{currentPage+1}} of {{numPages+1}}</div>
              <div class="column" style="text-align:right">
            <button class="button" v-if="isNextPage" @click="nextPage">Next</button>
          <button class="button" v-if="isNextPage" @click="lastPage">>></button>
            </div>
            </div>
        </section>
      </div>
    </div>
  </div>
</template>

<script>
import navbar from './ui/Navbar'
import itemcard from './ui/ItemCard'
import firebase from 'firebase'
import ThreeDots from 'vue-loading-spinner/src/components/ThreeDots'

export default {
  name: 'home',
  components: {
      navbar,
      itemcard,
      ThreeDots
  },
  data: function() {
    return {
      currentUser: {
          name: '',
          photo: '',
          items: []
      },
      sort: 'Popularity',
      searchText: '',
      minPrice: '',
      maxPrice: '',
      categories: [
        'None',
        'Fashion',
        'Technology',
        'Electronics',
        'Motors',
        'Home & Garden',
        'Sports',
        'Health & Beauty',
        'Toys & Media',
        'Collectables',
        'Other'
      ],
      itemType: 'unfulfilled',
      category: '',
      purchased: '',
      selectedStores: [],
      showFaves: false,
      originalList: [],
      paginate: ['items'],
      length: 0
    }
  },
  created: function() {
    // Redirect if no page query parameter is provided
    if (!this.$route.query.page) {
      this.$router.push({ path: 'home', query: { page: '0' }});
      location.reload();
    } else {

    this.currentUser.items = [];
    this.originalList = [];

    // Get current user ID and reference to database
    var userId = firebase.auth().currentUser.uid;
    var db = firebase.database();

    // Get the name for the current user
    db.ref('/users/' + userId).once('value').then((snapshot) => {
      var username = (snapshot.val() && snapshot.val().fname) || '';
      this.currentUser.name = username;
    });

    // Get the current user's items
    var ref = db.ref('/users/' + userId + '/items').on("value", (snapshot) => {
      this.currentUser.items = [];
      snapshot.forEach((child) => {
        var item = child.val();
        item.key = child.key;
        item.host = this.extractStoreName(item.link)
        this.currentUser.items.unshift(item);
        this.originalList.push(item);
      });
    }, function (errorObject) {
    });
  }
  },
  methods: {
    // Go to next page of items
    nextPage: function() {
      var next = parseInt(this.$route.query.page) + 1;
      this.$router.push({ path: 'home', query: { page: next }});
    },
    // Go to previous page of items
    prevPage: function() {
      var prev = parseInt(this.$route.query.page) - 1;
      this.$router.push({ path: 'home', query: { page: prev }});
    },
    // Go to last page of items
    lastPage: function() {
      var i = 0;
      var count = 0;
      while (i < this.length) {
        i = i + 8;
        count++;
      }
      count--;
      this.$router.push({ path: 'home', query: { page: count }});
    },
    // Go to first page of items
    firstPage: function() {
      this.$router.push({ path: 'home', query: { page: 0 }});
    },
    // Sorting function based on popularity of items
    popularitySort: function(list) {
      var storeList = [];
      var itemsLen = list.length;

      // items will be sorted.
      if (itemsLen != 0) {
        this.sort = 'Popularity'
      // items will not be sorted, this.currentUser.items.length is empty.
      } else {
        this.sort = 'Last added';
      };

      // Calculate store's popularity for every stores by number of appearances
      for (var i = 0; i < itemsLen; i++) {
        var inserted = false;
        if (list[i].link) {
          var storeName = this.extractStoreName(list[i].link);
          for (var j = 0; j < storeList.length; j++) {
            if (storeList[j].name == storeName) {
              storeList[j].rate++;
              inserted = true;
            };
          };
          if (!inserted) storeList.push({name:storeName, rate:1});
        };
      };

      // Calculate overall popularity (clicks + store's popularity)
      for (var i = 0; i < itemsLen; i++) {
        var storeName = this.extractStoreName(list[i].link);
        var popularity = list[i].clicks;
        for (var j = 0; j < storeList.length; j++) {
          if (storeList[j].name == storeName) popularity += storeList[j].rate;
        };
      };

      // Sort the items by overall popularity (clicks + store's popularity)
      list.sort(function(a, b) {return b.clicks - a.clicks});
      return list;
    },
    // Extract the hostname for a given url
    extractHostname: function (url) {
      var hostname;
      //find & remove protocol (http, ftp, etc.) and get hostname
      if (url.indexOf("://") > -1) {
          hostname = url.split('/')[2];
      } else {
          hostname = url.split('/')[0];
      }
      //find & remove port number
      hostname = hostname.split(':')[0];
      //find & remove "?"
      hostname = hostname.split('?')[0];

      return hostname;
    },
    // Extract the store name for a given url
    extractStoreName: function (url) {
      var name = this.extractHostname(url);
      var splitArr = name.split('.');
      var arrLen = splitArr.length;
      //extracting the root domain here
      //if there is a subdomain
      if (arrLen > 2) {
          name = splitArr[arrLen - 2] + '.' + splitArr[arrLen - 1];
          //check to see if it's using a Country Code Top Level Domain (ccTLD) (i.e. ".me.uk")
          if (splitArr[arrLen - 1].length == 2 && splitArr[arrLen - 1].length == 2) {
              //this is using a ccTLD
              name = splitArr[arrLen - 3] + '.' + name;
          }
      }
      return name;
    },
    resetFilters: function () {
      this.minPrice = '';
      this.maxPrice = '';
      this.category = '';
      this.showFaves = false;
      this.itemType = 'unfulfilled';
      this.selectedStores = [];
    },
    // Filters item list by search text (non case sensitive)
    filterSearch: function (list, itemList) {
      if (!this.searchText) {
        list = itemList;
      } else {

        var i, len, item;
        len = itemList.length;
        for (i = 0; i < len; i++) {
          item = itemList[i].title.toUpperCase()
          if (item.indexOf(this.searchText.toUpperCase()) !== -1) {
            list.push(itemList[i]);
          }
        }
      }
      return list;
    },
    // Filter items by if they have been favourited
    filterFavourites: function (list) {
      if (this.showFaves) {
        var i, len;
        var newList = [];
        len = list.length;
        for (i = 0; i < len; i++) {
           if (list[i].favourite) {
             newList.push(list[i]);
          }
        }
        return newList;
      } else {
        return list;
      }
    },
    // filters item list by min and max prices
    filterPrice: function (list) {
      var newList = [];
      if (!this.minPrice && !this.maxPrice) {
        return list;
      } else {
        var i, len, item;
        len = list.length;
        for (i = 0; i < len; i++) {
          item = parseInt(list[i].price);
          // if only min price is empty
          if (!this.minPrice) {
            if (item <= this.maxPrice) {
              newList.push(list[i]);
            }
          }
          // if only max price is empty
          else if (!this.maxPrice) {
            if (item >= this.minPrice) {
              newList.push(list[i]);
            }
          } else {
            if (item >= this.minPrice && item <= this.maxPrice) {
              newList.push(list[i]);
            }
          }
        }
        return newList;
      }
    },
    // filters item list by categories
    filterCategory: function (list) {
      var newList = [];
      if (!this.category || this.category == 'None') {
        return list;
      } else {
        var i, len, itemCat;
        len = list.length;
        for (i = 0; i < len; i++) {
          itemCat = list[i].category;
          if (itemCat === this.category) {
            newList.push(list[i]);
          }
        }
        return newList;
      }
    },
    // Filter by item type
    filterItemType: function (list) {
      var newList = [];
      var len = list.length;
      for (var i = 0; i < len; i++) {
        var purchased = list[i].purchased;
        if (this.itemType === 'unfulfilled' && !purchased) {
          newList.push(list[i]);
        } else if (this.itemType === 'fulfilled' && purchased) {
          newList.push(list[i]);
        } else if (this.itemType === 'all') {
          newList.push(list[i]);
        }
      }
      return newList;
    },
    // Filter items by which store they're from
    filterStores: function (list) {
      var newList = [];
      var storeListLength = this.selectedStores.length;
      if (this.selectedStores.length == 0) {
        return list;
      } else {
        var i, j, len, storeLink, listLink;
        len = list.length;
        for (i = 0; i < storeListLength; i++) {
          storeLink = this.selectedStores[i];
          for (j = 0; j < len; j ++) {
            listLink = list[j].link;
            if (!listLink) {
              continue;
            } else {
              if (this.extractStoreName(listLink) === storeLink) {
                newList.push(list[j]);
              }
            }
          }
        }
        return newList;
      }
   },
   // Sort items by the current "sort by" filter
   sortBy: function(list) {
      var temp = list;
      if (this.sort == 'First added') {
         list.reverse();
      } else if (this.sort == 'Popularity') {
         list = this.popularitySort(list);
      } else if (this.sort == 'Last added') {
         list = temp;
      } else if (this.sort == 'Price low to high') {
         list.sort(function(a, b) {
            return parseFloat(a.price) - parseFloat(b.price);
         });
      } else if (this.sort == 'Price high to low') {
         list.sort(function(a, b) {
             return parseFloat(b.price) - parseFloat(a.price);
         });
      }
      return list;
   },
   // Show only certain amount of items depending on page number
   showOnly: function (list) {
     var newList = [];
     var start = parseInt(this.$route.query.page)*8;
     var end = start + 8;
     newList = list.slice(start, end);
     return newList;
   },
  },
  computed: {
    // Number of pages of items in total
    numPages: function () {
      var i = 0;
      var count = 0;
      while (i < this.length) {
        i = i + 8;
        count++;
      }
      count--;
      return count;
    },
    // Current page number
    currentPage: function () {
      return parseInt(this.$route.query.page);
    },
    // If there is a next page
    isNextPage: function () {
      var amount = (parseInt(this.$route.query.page)+1)*8;
      return (this.filteredItems.length >= 8) && (amount < this.length);
    },
    // Return list of items with all filters applied
    filteredItems: function () {
      var list = [];
      list = this.filterSearch(list, this.currentUser.items);
      list = this.filterFavourites(list);
      list = this.filterPrice(list);
      list = this.filterCategory(list);
      list = this.filterItemType(list);
      list = this.filterStores(list);
      list = this.sortBy(list);
      this.length = list.length;
      list = this.showOnly(list);
      return list;
    },
    // Return list of stores user has items from
    myStores: function () {
      var storeList = [];
      var i, len, itemStore;
      len = this.currentUser.items.length;
      for (i = 0; i < len; i++) {
        if (!this.currentUser.items[i].link) {
          continue;
        }
        itemStore = this.extractStoreName(this.currentUser.items[i].link)
        // checks if already in list
        if (storeList.indexOf(itemStore) < 0) {
          storeList.push(itemStore)
        }
      }
      return storeList;
    },
    // Returns number of stores in filtered list
    numStoresFilteredList: function () {
      var storeList = [];
      var i, len, itemStore;
      len = this.filteredItems.length;
      for (i = 0; i < len; i++) {
        if (!this.filteredItems[i].link) {
          continue;
        }
        itemStore = this.extractStoreName(this.filteredItems[i].link)
        // checks if already in list
        if (storeList.indexOf(itemStore) < 0) {
          storeList.push(itemStore);
        }
      }
      return storeList.length;
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

.button.is-primary:hover, .button.is-primary.is-hovered {
    background-color: rgba(0, 211, 209, 0.70);
    border-color: transparent;
    color: #fff;
}

.button.is-primary {
   background-color: #00d3d1;
}

.add-item-btn {
  margin-right: 0px;
  background-color: #00d3d1;
  color: white;
  border: none;
}
.menu-label {
  color: #00d3d1;
}

.select:not(.is-multiple)::after {
    border: 1px solid #00d3d1;
    border-right: 0;
    border-top: 0;
    content: " ";
    display: block;
    height: 0.5em;
    pointer-events: none;
    position: absolute;
    transform: rotate(-45deg);
    width: 0.5em;
    margin-top: -0.375em;
    right: 1.125em;
    top: 50%;
    z-index: 4;
}

.input:focus, .input.is-focused, .input:active, .input.is-active, .textarea:focus, .textarea.is-focused, .textarea:active, .textarea.is-active {
    border-color: #00d3d1;
    box-shadow: 0 0 0 0.125em rgba(0, 209, 178, 0.25);
}

.filter-price {
  height: 30px;
  font-size: 13px;
  box-shadow: none;
}

.filter-radio {
  padding-bottom:10px;
}

.num-items {
  color: #313131;
  font-weight: 200;
  text-align: left;
  font-size: 15px;
  padding-top: 9px;
  padding-bottom: 15px;
}
</style>
