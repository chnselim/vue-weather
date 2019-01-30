<template>
  <div class="container">
    <div class="header">
      <img src="https://image.flaticon.com/icons/png/512/987/987964.png"/>
      <h2>Weather<span>Map</span>
        <!--<label>by<a href="www.github.com/chnselim">CS</a></label>-->
      </h2>
    </div>
    <div class="search-container" v-click-outside="hideSearchList">
      <input placeholder="Type your city..." v-model="cityName"
             @focus="searchedCityResponseList.length > 0 ? isSearchListOpen = true: null"
             @keyup="cityName ? null: isSearchListOpen = false"
             @keyup.enter="searchCityByName()"/>
      <button @click="searchCityByName()"><span class="lnr lnr-chevron-right"></span></button>
      <div class="city-list" v-if="isSearchListOpen">
        <h2>SEARCH RESULT</h2>
        <p v-if="!isSearchingForCities && searchedCityResponseList.length === 0">
          No city found.
        </p>
        <Loader v-if="isSearchingForCities" color="dark"></Loader>
        <div class="city" v-for="city of searchedCityResponseList">
          <p @click="addCityToList(city)"
             v-bind:class="{
                  'active': selectedCities.some(c => c.id === city.id)
                  }">{{city.name}}, {{city.sys.country}}
            <span v-bind:class="{
                  'lnr lnr-circle-minus red': selectedCities.some(c => c.id === city.id),
                  'lnr lnr-plus-circle green': !selectedCities.some(c => c.id === city.id)
                  }"></span>
          </p>
        </div>
      </div>
    </div>
    <template v-for="city of selectedCities">
      <div class="item"
           v-bind:class="{'expanded': city.expanded}"
           @click="getCityDetailByName(city)">
        <div class="item-header">
          <p class="city">{{city.name}}</p>
          <p class="degree">{{integer(city.max_degree)}}&#8451; / <span>{{integer(city.min_degree)}}&#8451;</span></p>
        </div>
        <div class="item-body">
          <Loader v-if="isSearchingForDetail && !city.weeklyWeatherStats"></Loader>
          <template v-for="(stat, index) in city.weeklyWeatherStats">
            <div class="day-card" v-if="index !== 0 && index % 8 === 0">
              <img v-if="stat.weather[0].main === 'Clear'" src="https://image.flaticon.com/icons/svg/979/979585.svg"/>
              <img v-if="stat.weather[0].main === 'Clouds'"
                   src="https://image.flaticon.com/icons/svg/1420/1420543.svg"/>
              <img v-if="stat.weather[0].main === 'Rain'" src="https://image.flaticon.com/icons/svg/861/861056.svg"/>
              <img v-if="stat.weather[0].main === 'Snow'" src="https://image.flaticon.com/icons/svg/1200/1200430.svg"/>
              <img v-if="stat.weather[0].main === 'Extreme'"
                   src="https://image.flaticon.com/icons/svg/1453/1453052.svg"/>
              <span class="degree">{{integer(stat.main.temp_max)}}&#8451;</span>
              <span class="date">{{momentJS(momentJS.unix(stat.dt)._d).format("DDMMM")}}</span>
            </div>
          </template>
        </div>
        <button class="remove-button" @click="deleteCity(city)"><span class="lnr lnr-cross"></span></button>
      </div>
    </template>
  </div>
</template>

<script>
  import Loader from './Loader';
  import moment from 'moment';
  import ClickOutside from 'vue-click-outside'

  const apiUrls = {
    base_uri: 'https://api.openweathermap.org/data/2.5/',
  };

  export default {
    name: 'Home',
    components: {Loader},
    data() {
      return {
        momentJS: moment,
        integer: parseInt,
        cityName: '',
        selectedCities: [],
        searchedCityResponseList: [],
        isSearchListOpen: false,
        isSearchingForCities: false,
        isSearchingForDetail: false
      }
    },
    created() {
      // todo get data from localstorage
    },
    methods: {
      searchCityByName() {
        this.isSearchingForCities = true;
        this.isSearchListOpen = true;
        this.searchedCityResponseList = [];
        const uri = apiUrls.base_uri + 'find?q=' + this.cityName + '&units=metric&appid=cba6ffd3e56cb74c9a4a52f1e338fd84';
        fetch(uri)
          .then(response => response.json())
          .then(data => {
            data.list ? this.searchedCityResponseList = data.list : this.searchedCityResponseList = [];
            this.isSearchingForCities = false;
          })
          .catch(err => this.isSearchingForCities = false);
      },
      addCityToList(city) {
        const newCity = {
          id: city.id,
          name: city.name,
          country: city.sys.country,
          max_degree: city.main.temp_max,
          min_degree: city.main.temp_min,
          time: city.dt,
          expanded: false,
          weeklyWeatherStats: null
        };
        if (!this.selectedCities.some(city => city.id === newCity.id)) {
          this.selectedCities.push(newCity);
        } else {
          this.selectedCities.splice(this.selectedCities.findIndex(c => c.id === newCity.id), 1);
        }
      },
      getCityDetailByName(city) {
        this.isSearchingForDetail = true;
        this.selectedCities.forEach(c => c !== city ? c.expanded = false : null);
        city.expanded = !city.expanded;
        const uri = apiUrls.base_uri + 'forecast?q=' + city.name + ',' + city.country + '&units=metric&APPID=cba6ffd3e56cb74c9a4a52f1e338fd84';
        fetch(uri)
          .then(response => response.json())
          .then(data => {
            city.weeklyWeatherStats = data.list;
            this.isSearchingForDetail = false;
          });
      },
      deleteCity(city) {
        this.selectedCities.splice(this.selectedCities.findIndex(c => c.id === city.id), 1);
      },
      hideSearchList() {
        this.isSearchListOpen = false
      }
    },
    directives: {
      ClickOutside
    }
  }
</script>

<style lang="scss" scoped>
  .search-container {
    position: relative;
    display: flex;
    align-items: center;
    input {
      width: 320px;
      height: 60px;
      border: none;
      border-radius: 10px;
      box-shadow: 0 1px 0 #c1c1c1;
      outline: none;
      font-size: 25px;
      font-weight: 100;
      padding: 0 24px;
      margin-bottom: 3px;
      @media (max-width: 450px) {
        width: 260px;
      }
      &:focus ~ button {
        background: rgba(56, 56, 56, 0.03);
      }
    }
    button {
      position: absolute;
      right: 20px;
      font-weight: 500;
      border: none;
      color: #555555;
      border-radius: 7px;
      width: 32px;
      height: 32px;
    }
    .city-list {
      position: absolute;
      z-index: 1;
      left: 0;
      min-height: 124px;
      height: auto;
      top: 64px;
      display: block;
      width: 320px;
      background: 0 1px #ececec;
      border-radius: 9px;
      padding: 0 24px 15px 24px;
      -webkit-box-shadow: 0 1px 0 #e0e0e0;
      box-shadow: 0 1px 0 #e0e0e0;
      @media (max-width: 450px) {
        width: 260px;
      }
      h2 {
        font-weight: 500;
        font-size: 15px;
        border-bottom: 1px solid rgba(227, 227, 227, 0.56);
        padding-bottom: 8px;
        color: gray;
      }
      .city {
        p {
          background: #32304a;
          color: white;
          padding: 13px 12px;
          border-radius: 10px;
          display: flex;
          justify-content: space-between;
          margin: 8px 0;
          cursor: pointer;
          &:hover, &.active {
            background: #1e1d2c;
          }
          span {
            &.red {
              color: #ff7979;
            }
            &.green {
              color: #00ff7f;
            }
          }
        }
      }
    }
  }

  .container {
    display: flex;
    align-items: center;
    flex-direction: column;
    margin-top: 30px;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    .header {
      display: inline-flex;
      align-items: center;
      img {
        width: 50px;
        height: 50px;
        opacity: .2;
        transition: all .3s;
      }
      h2 {
        font-size: 30px;
        font-weight: 900;
        text-transform: uppercase;
        color: #e2e2e2;
        position: relative;
        transition: all .3s;
        span {
          font-weight: 300;
        }
        label {
          font-size: 15px;
          font-weight: 300;
          text-transform: initial;
          a {
            text-decoration: none;
            color: inherit;
          }
        }
      }
      &:hover {
        h2 {
          transition: all .3s;
          color: #3c3c3c;
          a {
            transition: all .3s;
            color: #ff1001;
          }
        }
        img {
          transition: all .3s;
          opacity: 1;
        }
      }
    }
    .item {
      position: relative;
      display: block;
      width: 320px;
      height: 60px;
      background: 0 1px #5c2dff;
      border-radius: 10px;
      padding: 0 24px;
      margin: 1px 0;
      box-shadow: 0 1px 0 #e0e0e0;
      cursor: pointer;
      transition: all .3s;
      @media (max-width: 450px) {
        width: 260px;
      }
      &.expanded {
        background: #4a25cd;
        height: 180px;
        .item-body {
          display: flex;
          flex-direction: row;
          justify-content: center;
        }
        .remove-button {
          display: block;
        }
      }
      .item-header {
        display: flex;
        justify-content: space-between;
        p {
          color: white;
          font-size: 20px;
          margin: 0;
          margin-top: 18px;
          text-transform: uppercase;
          &.city {
          }
          &.degree {
            font-weight: 600;
            span {
              font-weight: 100;
              font-size: 18px;
            }
          }
        }
      }
      .item-body {
        display: none;
        margin: 7px 0;
        padding-top: 10px;
        border-top: 1px solid rgba(255, 255, 255, 0.78);
        .day-card {
          display: flex;
          justify-content: flex-start;
          align-items: center;
          flex-direction: column;
          width: 72px;
          height: 110px;
          border-radius: 8px;
          margin: 0 5px;
          background: rgba(0, 0, 0, 0.2);
          &:first-child {
            margin-left: 0;
          }
          &:last-child {
            margin-right: 0;
          }
          img {
            margin-top: 10px;
            width: 30px;
          }
          .degree {
            color: #fff;
            font-size: 20px;
            margin: 8px 0;
          }
          .date {
            margin-top: 5px;
            color: #c3c3c3;
            font-size: 16px;
            text-transform: uppercase;
          }
        }
      }
      .remove-button {
        display: none;
        position: absolute;
        right: -28px;
        top: 10px;
        height: 40px;
        width: 28px;
        background: #ff2e6d;
        border: none;
        border-radius: 25px;
        border-top-left-radius: 0;
        border-bottom-left-radius: 0;
        transition: all .3s;
        span {
          color: white;
          font-weight: 900;
          font-size: 14px;
          margin-left: -6px;
        }
      }
    }
  }
</style>
