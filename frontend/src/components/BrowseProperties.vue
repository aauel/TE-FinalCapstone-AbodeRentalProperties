<template>
  <div class="container is-fluid">
    <div class="center">
        <h1 class="title">Find your next apartment!</h1>
    </div>
    <div id="forms">
        <form id="search">
            <b-field label="Search by Address or Name" label-position="inside">
                <b-input rounded type="text" id="textSearch" v-model="textSearch" icon="magnify"></b-input>
                <p class="control">
                    <b-button @click.prevent="search" type="is-primary" >Search</b-button>
                </p>
            </b-field>
        </form>
        <form id="filters">
                <b-field label="Bed Count" label-position="inside">
                    <b-select @input="filter" id="filterBr" v-model="brFilter">
                        <option value=""></option>
                        <option value="0">Studio</option>
                        <option value="1">1 Br</option>
                        <option value="2">2 Br</option>
                        <option value="3">3 Br</option>
                        <option value="4">4 Br</option>
                    </b-select>
                </b-field>
                <b-field label="Max Rent" label-position="inside">
                    <b-select @input="filter" id="filterBr" v-model="upperRentFilter">
                        <option value=""></option>
                        <option value="500">$500/mo</option>
                        <option value="750">$750/mo</option>
                        <option value="1000">$1000/mo</option>
                        <option value="1500">$1500/mo</option>
                        <option value="2000">$2000/mo</option>
                    </b-select>
                </b-field>
                <b-field label="Neighborhood" label-position="inside">
                    <b-select @input="filter" id="filterLocation" v-model="locationFilter">
                        <option value=""></option>
                        <option>Downtown</option>
                        <option>East Side</option>
                        <option>West Side</option>
                        <option>North Side</option>
                        <option>NKY</option>
                    </b-select>
                </b-field>
                <!-- <b-button @click.prevent="filter" type="is-primary" expanded>Filter</b-button> -->
        </form>
    </div>
    <div id="properties">
        <div v-for="property in shownProperties" :key="property.propertyId">
           <b-collapse class="card" animation="slide">
                <div slot="trigger" slot-scope="props" class="card-header" role="button">
                   <p v-if="property.streetAddress !== property.propertyName" class="card-header-title">{{property.propertyName}} @ {{property.streetAddress}}</p>
                   <p v-if="property.streetAddress === property.propertyName" class="card-header-title">{{property.streetAddress}}</p>
                   <a class="card-header-icon">
                        <b-icon
                            :icon="props.open ? 'menu-up' : 'menu-down'">
                        </b-icon>
                    </a>
                </div>
                    <div class="card-content">
                        <div class="tile is-ancestor">
  <div class="tile is-4 is-parent">
    <div class="tile is-child">
         <img v-if="property.photoPath.includes('http')" :src="property.photoPath" class="property-img">
                        <img v-if="property.photoPath.includes('http') === false" class="property-img" :src="require('@/assets/images/properties/' + property.photoPath)">
    </div>
   
  </div>
  <div class="tile is-vertical is-parent">
    <div class=" detailContainter tile is-child box has-background-white-ter">
        <div>
        <i class="fas fa-map-marker-alt fa-2x"></i>
        </div>

     <div class="">
                            <p><strong>Neighborhood:</strong> {{property.location}}</p>
                            <p><strong>City:</strong> {{property.city}}</p>
                            <p><strong>State:</strong> {{property.state}}</p>
                        </div>
                        <div><i class="far fa-building fa-2x"></i></div>
                        <div>
                                  <p><strong>Units:</strong></p>
                            <ul v-if="property.units.length > 0">
                                <li v-for="unit in property.units" :key="unit.unitId">
                                    <span :class="'available-' + unit.available">#{{unit.unitNumber}}: ({{unit.bedCount}} Bed/ {{unit.bathCount}} Bath) for ${{unit.price}}/mo</span>
                                </li>
                            </ul>
                        </div>
    </div>
     <div class="tile is-child box has-background-white-ter">
      <p><strong>Features:</strong></p>
                    <ul class="featureList">
                        <li v-for="feature in property.features" :key="feature.featureId"> {{feature.featureName}}</li>
                        </ul>
    </div>
  </div>
</div>
                    </div>         
                <footer class="card-footer">
                     <b-button @click="toast" type="is-primary" expanded v-if="property.streetAddress !== property.propertyName"> <i class="fas fa-angle-double-right"></i>Apply to {{property.propertyName}} @ {{property.streetAddress}} </b-button>
                   <b-button @click="toast" type="is-primary" expanded v-if="property.streetAddress === property.propertyName" > <i class="fas fa-angle-double-right"></i>Apply to {{property.streetAddress}}</b-button>
                </footer>
           </b-collapse> 
        </div>    
    </div>    
  </div>
</template>

<script>
// import auth from '../auth';
export default {
    data() {
        return {
            allProperties: [],
            shownProperties: [],
            textSearch: '',
            brFilter: '',
            upperRentFilter: '',
            locationFilter: '',
            props: {
                open: true
            },
        }
    },
    props: {
        user: Object
    },
    methods: {
        toast(){
            if(this.user != null) {
                this.$buefy.toast.open({
                    message: 'Application Sent!',
                    type: 'is-success'})
            } else {
                this.$buefy.toast.open({
                    message: 'Please Log In!',
                    type: 'is-danger'})
            }
        },
        search() {
            console.log('searching');
            this.shownProperties = this.allProperties.filter(p => {
                let nameAndAddress = p.propertyName + p.streetAddress;
                nameAndAddress = nameAndAddress.toLowerCase();
                return nameAndAddress.includes(this.textSearch.toLowerCase())
            });
        },
        filter() {
            console.log('filtering');
            this.shownProperties = this.allProperties;
            if(this.brFilter !== '') {
                this.shownProperties = this.shownProperties.filter(p => {
                    let result = false;
                    p.units.forEach(u => {
                        if(u.bedCount == this.brFilter) {
                            result = true;
                        }
                    });
                    return result;
                });
            }
            if(this.upperRentFilter !== '') {
                this.shownProperties = this.shownProperties.filter(p => {
                    let result = false;
                    p.units.forEach(u => {
                        if(u.price <= parseInt(this.upperRentFilter)) {
                            result = true;
                        }
                    });
                    return result;
                });
            }
            if(this.locationFilter !== '') {
                this.shownProperties = this.shownProperties.filter(p => {
                    return p.location === this.locationFilter;
                });
            }
        },
        pathToPhoto(photoPath) {
            return '@/assets/images/properties/' + photoPath;
        }
    },
    created() {
        fetch('http://localhost:8080/api/properties')
        .then (response => {
            if(response.ok) {
                return response.json();
            }
        }).then(responseData => {
            this.allProperties = responseData;
            this.shownProperties = responseData;
        });
    }

}
</script>

<style scoped>
    .center {
        text-align: center;
    }
    h1, #search {
        margin-bottom: 40px;
    }
    i {
        padding-right: 10px;
    }
    #forms {
        width: 100%;
        display: flex;
        justify-content: center;
    }
    #filters {
        margin-left: 10px;
        display: flex
    }
    /* #properties {
        display: flex;
        flex-wrap: wrap;
        justify-content: flex-start;
    } */
    /* #properties div {
        width: 100%;
    } */
   .property-img {
         object-fit: cover;
         height: 100%;
         border-radius: 8px;
    }
    img {
        width: 100%;
      object-fit: cover;
    }
    li {
        list-style: none;
    }
    span.available-false {
        text-decoration: line-through;
    }
    .content:nth-child(n) {
        width: 30%;
    }
    .content {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        justify-content: flex-start;
    }
.featureList {
     -moz-column-count: 3;
    -moz-column-gap: 20px;
    -webkit-column-count: auto;
    -webkit-column-gap: 20px;
    column-count: 3;
    column-gap: 20px;
}
.detailContainter {
        display: flex;
        flex-direction: row;
        justify-content: space-around;
        align-items: center;
}
#properties div {
    margin-bottom: 25px;
}

</style>