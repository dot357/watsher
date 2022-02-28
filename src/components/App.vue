<template>
  <Page actionBarHidden="true">
 

 


    <GridLayout v-if="!show.dockMenu" cols="auto,auto, auto" rows="45,*, 75" backgroundColor="#9DC9D9" >    
     <SearchBar v-if="show.searchBar" ref="searchBar" col="0" row="0" hint="Please enter a city"  v-model="city"  @submit="searchForCity" /> 
 
     <GridLayout col="0" row="1"  cols="auto, auto" rows="75,*" >
        <SegmentedBar col="0" row="0" backgroundColor="transparent" selectedBackgroundColor="#0A2835" >
        <SegmentedBarItem :title="currentCity ? currentCity : 'Loading'" fontWeight="bold"   /> 
        
      </SegmentedBar> 


       <Label col="0" row="1" ref="weather" :text="weather ? (weather.main.temp - 273.5).toFixed() : 'Wathser'"  fontSize="18" padding="20" />
     </GridLayout>


     <FlexBoxLayout col="0" row="2" width="*" backgroundColor="#0A2835" alignItem="center" justifyContent="center">
        <Button text="Menu" @tap="show.dockMenu = true" /> 
       <Button :text="!show.searchBar ? 'Search' : 'Close'" @tap="show.searchBar = !show.searchBar" />
        <Button text="Locate me" @tap="getCurrentLocation" /> 
     </FlexBoxLayout> 
    
 
    
  
    </GridLayout>  

   
<DockLayout v-if="show.dockMenu" class="dock-menu" stretchLastChild="false">

  <GridLayout cols="auto, auto, auto" rows="55, *,auto" dock="left" width="60%" class="dock-menu-drawer">
    <Image src="~/assets/images/logo-white.png"  col="0" row="0" stretch="aspectFit" class="logo" />
  <ListView for="item in localStorageCities.slice().reverse()"  col="0" row="1"   @itemTap="clickNloadWeather">
  <v-template> 
    <Label text="Close" /> 
    <!-- Shows the list item label in the default color and style. -->
    <Label :text="item" class="menu-item" />
  </v-template> 
</ListView>

<FlexBoxLayout col="0" row="2" alignItem="center" justifyContent="center">
  <Button text="Clear"   @tap="clearFavList" />
  <Button text="Close"   @tap="show.dockMenu = false" />
</FlexBoxLayout>


  </GridLayout>


    
</DockLayout>


    
  </Page>
</template>

<script>
import * as http from "http";
import * as geolocation from "nativescript-geolocation";
import { Accuracy } from "tns-core-modules/ui/enums";
import * as appSettings from "tns-core-modules/application-settings"

//import  moment from "moment"
export default {
  data() {
    return {
      loading : {
        weather : false
      },
      show : {
        searchBar : false,
        dockMenu : false,
      },
      locationMetrics : null,
      city: null,
      currentCity : null,
      weather: null,
      currentLocation: null,
      localStorageCities : []
      
    };
  },
  async created() {

   
    if(appSettings.getString("cities")) this.localStorageCities = JSON.parse(appSettings.getString("cities"))
  

    geolocation.enableLocationRequest();
    await this.getCurrentLocation();
    console.log("created")

    
   
   
  },
  async mounted() {
    await this.getCurrentLocation();

   
  },
  methods: {
    async getCurrentLocation(){
        this.currentLocation = await geolocation.getCurrentLocation({
              desiredAccuracy: Accuracy.high,
              maximumAge: 5000,
              timeout: 20000,
            });
      await this.Inıt();
      await this.getWeather();
    },
    async Inıt() {
      const apiKey = "6637a91601b008dc52f0e1d1d72d9d93";
      if (this.currentLocation) {
        await http
          .getJSON(
            `https://api.openweathermap.org/geo/1.0/reverse?lat=${this.currentLocation.latitude}&lon=${this.currentLocation.longitude}&limit=5&appid=${apiKey}`
          )
          .then((res) => {
           
            this.locationMetrics = res;
            this.city = res[0].name;
            
          })
          .catch((err) => {   
            console.log(err);
          });
      }
    },
    
    async getWeather() {
      this.weather = null;
      const apiKey = "6637a91601b008dc52f0e1d1d72d9d93";
      let url;

      url = `https://api.openweathermap.org/data/2.5/weather?q=${this.city}&appid=${apiKey}`;
      
      await http
        .getJSON(url)
        .then((result) => {
          this.weather = result;
          //console.log(this.weather);
          this.currentCity = this.city
          /* 
          TODO : If city cannot be found use state instead
          */
        
         
          
        })
        .catch((err) => {
          console.log(err);
        }); 
    }, 
    async searchForCity(){
      await this.getWeather()
      //dismiss the keyboard
      this.$refs.searchBar.nativeView.dismissSoftInput();


     

        if(this.weather.name){
          let firstLetterUppercase = this.city.charAt(0).toUpperCase() + this.city.slice(1);
          console.log(this.localStorageCities)
          this.localStorageCities.push(firstLetterUppercase)
          
          let removeDublicates = [...new Set(this.localStorageCities)];
          this.localStorageCities = removeDublicates
        console.log(this.localStorageCities)
        appSettings.setString("cities", JSON.stringify(removeDublicates))  
        }
        
      
      this.show.searchBar = false

 
   
    },
    async clickNloadWeather($e){
      console.log($e.item)
      this.city = $e.item
      await this.getWeather()
      this.show.dockMenu = false
    },
    clearFavList(){
      this.localStorageCities = []
      appSettings.remove("cities")
      this.show.dockMenu = false
    }
  },
};
</script>

<style scoped>

:root{
  --dark-blue : #0A2835;
}
ActionBar {
  background-color: #0a2835;
  color: #ffffff;
}

.message {
  vertical-align: center;
  text-align: center;
  font-size: 20;
  color: #333333;
}

.text-white {
  color: white;
}


.dock-menu{
  background-color : rgba(0,0,0,0.8);
}

.dock-menu-drawer{
  background-color: #0A2835;
  color : white;
  padding: 20;
}

.menu-item{
 border:none;
 
}

.logo{
 
}
</style>
 