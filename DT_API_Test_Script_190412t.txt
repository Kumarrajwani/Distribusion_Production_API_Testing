 //------------Mandatory Inputs (Departure, Arrival stations and Departure date)-----------------------------------
 //------------Optional Inputs (Currency & Number of passengers(pax))----------------------------------------------
 
pm.globals.set('departure_stations[]','GBLONLLS');

pm.globals.set('arrival_stations[]','GBLONSFB');
 
pm.globals.set('departure_date','2019-04-16');

pm.globals.set('currency','USD');
    
pm.globals.set('pax','2');



	
	
	
	
--------------------------- Output  (Price comparison and Vacancy check)-----------------Test Tab----------------
	//Fetching & Storing all prices from C#F
var jsonData = JSON.parse(responseBody);
var Body =  jsonData.data;
var result1 = [];
var j=0;
for(i=0;i<Body.length;i++){
    result1[j] = Body[i].attributes.cheapest_total_adult_price;
    j=j+1;
    }
    
pm.globals.set("Price", JSON.stringify(result1));
var Price = JSON.parse(pm.globals.get("Price"));

//Fetching & Storing all departure times from C#F
var result2 = [];
var j=0;
for(i=0;i<Body.length;i++){
    result2[j] = Body[i].attributes.departure_time;
    j=j+1;
    }

pm.globals.set("Departure_time", JSON.stringify(result2));
var Departure_time = JSON.parse(pm.globals.get("Departure_time"));

//Fetching & Storing all arrival times from C#F 
var result3 = [];
var j=0;
for(i=0;i<Body.length;i++){
    result3[j] = Body[i].attributes.arrival_time;
    j=j+1;
    }

pm.globals.set("Arrival_time", JSON.stringify(result3));
var Arrival_time = JSON.parse(pm.globals.get("Arrival_time"));

//Fetching & Storing all marketing carries from C#F 
var result4 = [];
var j=0;
for(i=0;i<Body.length;i++){
    result4[j] = Body[i].relationships.marketing_carrier.data.id;
    j=j+1;
    }

pm.globals.set("Marketing_carrier", JSON.stringify(result4));
var Marketing_carrier = JSON.parse(pm.globals.get("Marketing_carrier"));
var jsonData = JSON.parse(responseBody);
    pm.globals.set('CF_pax', jsonData.included[1].relationships.passenger_types.data[0].id);
var departure_station = pm.globals.get("departure_stations[]");
    pm.globals.set('departure_station',departure_station);
var arrival_station = pm.globals.get("arrival_stations[]");
    pm.globals.set('arrival_station',arrival_station);
var CF_pax = pm.globals.get("CF_pax");
    pm.globals.set('CF_pax', CF_pax);
var currency = pm.globals.get("currency");
    pm.globals.set('currency', currency);
var pax = pm.globals.get("pax");
    pm.globals.set('pax', pax);
    
pm.test("Total trips for the route (" +departure_station + "-"+arrival_station+")"+": " +Departure_time.length);

var myNumber=0;
for(k=0;k<Departure_time.length; k++ ){
  pm.sendRequest({ 
    url: 'https://api.distribusion.com/retailers/v4/connections/vacancy?marketing_carrier=' +Marketing_carrier[k] +'&departure_station='+departure_station+'&arrival_station='+arrival_station+'&departure_time=' +Departure_time[k]+'&arrival_time=' + Arrival_time[k] +'&currency='+currency+'&passengers[][pax]='+pax+'&passengers[][type]=' + CF_pax,
    method: 'GET',
    header: 'Api-Key:8mdFIDMXn73CVgwwiAbWssV8Y8Fscfe5R5VMqVkq',
    
    },function(err, response){
         myNumber++;
         var totalResponse=response.json();
         
         var PriceArrayNumber=parseInt(myNumber-1);
        
         pm.test("Departure time: " + Departure_time[PriceArrayNumber] + " & Arrival time: " + Arrival_time[PriceArrayNumber]);
         pm.test("Curreny:" + currency + " & Number of passengers:" + pax);
         pm.test("C#F Price: " + Price[PriceArrayNumber] + " & C#V Price: " + totalResponse.data.attributes.total_price);
 
         
         if(totalResponse.data.attributes.total_price==Price[PriceArrayNumber]) {
            pm.test("Price matched");
        }
        else {
            pm.test("",function(){throw new Error("Price not matched" );});
        
        }
        if(totalResponse.data.attributes.vacant == true)
            pm.test("Vacancy found");
        else 
            pm.test("",function(){throw new Error("Vacancy not found" );});
       
    });
    }
    


    
    
