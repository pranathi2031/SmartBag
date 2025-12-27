<script>
  import { Router, Route, link } from 'svelte-routing';
  import bagImage from './lib/Shoulder-Bag-PNG-Isolated-HD.png';
  import { onMount } from 'svelte';
  import BagLoc from './BagLoc.svelte';
  import sound from './lib/whatsapp_ringtone.mp3';
  import { onDestroy } from 'svelte';

  let showTeam = false;
  let audio;
  let isShaking = false;
  let currentScreen ='home';
  let items=['laptop','lipbalm','keys','mobile','lipstick'];
  let bagItemLoc =['front pocket','middle pocket','secure compartment','side pocket','big compartment'];
  const criteriaItems = {
    "Examination Hall": ["admit card", "pen", "pencil", "eraser", "id card"],
    "International Travel": ["passport", "visa", "boarding pass", "mobile"],
    "Day trip": ["water bottle", "sunglasses", "snacks", "first aid kit", "camera","mobile","cash","credit card"],
    "College": ["books", "laptop", "id card", "notebooks"],
    "Shopping": ["wallet", "shopping list", "reusable bags", "coupons"],
    "Party": ["gift", "invitation","makeup kit", "accessories"]
  };
  let items_loc={
    "laptop":"middle compartment",
    "lipbalm":"front small compartment",
    "keys":"inner secure compartment",
    "mobile":"front big compartment",
    "lipstick":"front small compartment"
  }

  let alerts =new Set();
  let newItem = '';
  let newItem1 ='';
  let removeItem = false;
  let item_remove = '';
  let showInput = false;
  let showInput1 = false;
  let addItemNotes ='';
  let addItem = false;
  let checkedItems = new Set(); // Set to keep track of checked items
  let setImp = false;
  let connection=false;
  let lock = true;
  let isBagShaking = false;
  let criteria = '';
  let missingElements = [];
  $: selectedItems = criteria ? criteriaItems[criteria] : [];

function items_location(){
currentScreen = "itemsLocation"
items_loc ={...items_loc};
}
function stuck_zipper(){
addItemNotes = " Zipper Stuck!!!! - Adjusting items inside!!"
addItem = true;
}
function toggleItem(item) {
  if (checkedItems.has(item)) {
    checkedItems.delete(item); // Remove item if already checked
    alerts.add(`${item} removed from important items list!!!`)
  } else {
    checkedItems.add(item); // Add item if not checked
  }
  checkedItems = checkedItems;
  important_items();
  console.log(checkedItems);
}
  // Function to play alarm sound and trigger shake animation
  function triggerAlarm() {
    if (!isShaking) {
      isShaking = true;

      // Play the alarm sound
      audio = new Audio(sound);
      audio.play();

      // Stop the shake effect after 3 seconds (or duration of alarm)
      setTimeout(() => {
        isShaking = false;
      }, 10000);
    }
  }
  function triggerBag(audio) {
    if (!isBagShaking) {
      isBagShaking = true;

      // Play the alarm sound
      if(audio === true){
        audio = new Audio(sound);
        audio.play();
      }
      
      // Stop the shake effect after 10 seconds (or duration of alarm)
      setTimeout(() => {
        isBagShaking = false;
        if(audio === false){
        addItemNotes = "Check Zip Now !!!";
        addItem = true;
        }
        
      }, 10000);
    }
  }
  function team() {
    showTeam = !showTeam;
  }
  function connectBag(){
  currentScreen ="screen1";
  connection= true;
  }
  function disconnect(){
    currentScreen ="home";
    connection = false;
  }
  function toggle(){
    showInput = !showInput;
  }
  function toggle_remove(){
    showInput1 =!showInput1;
  }
  function customize(){
    currentScreen ="customization";
    important_items();
  }
  function add_item() {
    items = items.map(item => item.toLowerCase());
    if (newItem.trim() !== '' && !items.includes(newItem.toLowerCase())) { // Check if input is not empty
      items.push(newItem.trim().toLowerCase()); // Add new item
      addItemNotes = `${newItem.toLowerCase()} added to the bag`; // Confirmation message
      alerts.add(newItem.toLowerCase() + " added to the bag!!!");
      addItem = true; // Show confirmation message
      showInput = false; // Hide input after adding
      if(currentScreen ==='screen1')
      {
      items= items;
      }
      important_items();
      console.log(missingElements);
      const randomItem = bagItemLoc[Math.floor(Math.random() * bagItemLoc.length)];
      items_loc[newItem] = randomItem;
      items_loc = items_loc;
      console.log(items_loc);
      newItem='';
      
    }
    else if(newItem.trim()!==""){
      addItemNotes = `${newItem.toLowerCase()} is already in the bag`;
      newItem ='';
      addItem = true;
      showInput = false;
    }
    
  }
  function remove_item(){
  items = items.map(item => item.toLowerCase());
  newItem1 = newItem1.toLowerCase();
  if(newItem1.trim() !=='' && items.includes(newItem1)){

    items=items.filter(item => item.toLowerCase() !==newItem1.toLocaleLowerCase());

    addItemNotes = `${newItem1.toLowerCase()} removed from the bag`;
    alerts.add(`${newItem1.toLowerCase()} removed from the bag`);
    addItem = true;
    showInput1= false;
    important_items();
    alerts=alerts;
    console.log(missingElements);
    console.log(newItem1);
    if (newItem1 in items_loc) {
    delete items_loc[newItem1];
    }
    items_loc = {...items_loc};
    newItem1 = '';
    console.log(items_loc);
  }
  else if(newItem1.trim() !==''){
    addItemNotes = `${newItem1.toLowerCase()} is not present in the bag`
    newItem1 = '';
    addItem = true;
    showInput1 = false;
  }
  
  }
  function locSet(){
    currentScreen ="location";
  }
  function batteryStat(){
    currentScreen ="battery";
  }
  function homeStat(){
    currentScreen = "screen1";
  }
  function lockStat(){
    currentScreen ="lockStatus"
  }
  function lockBag(){
    lock = true;
  }
  function unlockBag(){

    lock = false;
  }
  function important_items(){
  missingElements = [...checkedItems].filter(item => !items.includes(item.toLowerCase()));
  console.log(missingElements);
  if(missingElements.length >0){
  missingElements.forEach(element => {
  alerts.add(`You dont have ${element} in the bag`);
  });
   
  console.log(alerts);
  }
  alerts=alerts;
  
  }
  function notifications(){
    currentScreen ="alerts";
  }
  
  function closeAlert(alert) {
  alerts.delete(alert);
  alerts = new Set(alerts);
}



  // Initialize the battery level (in percentage)
let batteryLevel = 100;
let isCharging = false;
let interval, chargingInterval;
let battery_icon = "fa fa-battery-full";
let remove_charger = false;
 

// Function to decrease the battery level
function startBatteryDrain() {
  if (interval) clearInterval(interval); // Clear any existing intervals
  interval = setInterval(() => {
    if (batteryLevel > 0 && !isCharging) {
      batteryLevel -= 1; // Decrease battery by 1%
    } else if (batteryLevel <= 0) {
      clearInterval(interval); // Stop draining when battery is empty
    }
    getBatteryIcon();
  }, 9000); // Decrease every 9 seconds
  getBatteryIcon();
}

// Function to stop the interval when the component is destroyed
onDestroy(() => {
  clearInterval(interval);
  clearInterval(chargingInterval);
});

// Function to start charging the battery
function chargeBattery() {
  isCharging = true;
  clearInterval(interval); // Stop draining when charging
  chargingInterval = setInterval(() => {
    if (batteryLevel < 100) {
      batteryLevel += 1; // Increase battery by 1%
    } else {
      clearInterval(chargingInterval); // Stop charging when full
      remove_charger = true;
    }
    getBatteryIcon();
  }, 9000); // Charge every 9 seconds
  addItemNotes = "Bag Charging";
  addItem = true;
}

// Function to unplug the battery
function unPlugBattery() {
  isCharging = false;
  remove_charger = false;
  clearInterval(chargingInterval); // Stop charging when unplugged
  addItemNotes = "Charger Unplugged";
  addItem = true;
  startBatteryDrain();
}

// Function to get the battery icon based on the level
function getBatteryIcon() {
  if (batteryLevel > 75) {
    battery_icon = "fa fa-battery-full"; // Full battery
  } else if (batteryLevel > 50) {
    battery_icon = "fa fa-battery-three-quarters"; // 3/4 battery
  } else if (batteryLevel > 25) {
    battery_icon = "fa fa-battery-half"; // Half battery
  } else if (batteryLevel > 10) {
    battery_icon = "fa fa-battery-quarter"; // Low battery
  } else {
    battery_icon = "fa fa-battery-empty"; // Critical battery
  }
}

// Start battery drain on component mount
startBatteryDrain();
</script>

<main>
  <header><h1>Smart Bag</h1></header>
  <div>
    <button class="normalButton" on:click={team}>Team Members</button>
    {#if showTeam}
    <div class="team-members">
      <ul>
        <li>Ana</li>
        <li>Harshini</li>
        <li>Pranathi</li>
        <li>Seethala</li>
      </ul>
    </div>
    {/if}
  </div>

  <br>
  <button class="normalButton"><a use:link href="https://docs.google.com/document/d/1dVW8rBgMPavoAOLUEzK2H2q5pW658IhKJmyi9GZ1Bj4/edit?usp=drive_link" target="_blank" class="doc">Documentation</a></button>
  <br><br>
  <button class="normalButton"><a use:link href="https://docs.google.com/document/d/1aLPBpAd9Q12k9djEQu4AL2HtJM27-qZYzTDHhFNS5oo/edit?usp=drive_link" target="_blank" class="doc">Info</a></button>
  <br><br>
  <div class="container">
    <div class="bag-screen">
      <div class="icons">
      {#if isCharging === true}
      <i class="fas fa-plug" style="color:green;"></i>
      {:else}
      <span class="icon-overlay">
        <i class="fas fa-plug" style="color:white; position:relative; left:3px;"></i>
        <i class="fas fa-ban" style="position: absolute; color: red;left:1px; right:2px; opacity:0.4;"></i>
      </span>
      {/if}
      {#if batteryLevel<=20}
      <button class="icon-button"><i class={battery_icon} style="color:red;"></i></button>
      {:else if isCharging}
      <button class="icon-button"><i class={battery_icon} style="color:green;"></i></button>
      {:else}
      <button class="icon-button"><i class={battery_icon} style="color:white;"></i></button>
      {/if}
      <span style="color:white; position:relative;right:6%;bottom:4%;"> {batteryLevel}%</span>
      {#if lock===true}
      <button class="icon-button" on:click={unlockBag}><i class="fas fa-lock" style="color:red;"></i></button>
      {:else}
      <button class="icon-button" on:click={lockBag}><i class="fa" style="color:green;">&#xf09c;</i></button>
      {/if}
      {#if connection === true}
      <button class="icon-button" on:click={triggerAlarm}><i class="material-icons" style="margin-top:5px; color:green; position:relative; top:4px;">phonelink_ring</i></button>
      {:else}
      <button class="icon-button"><i class="material-icons" style="margin-top:5px; color:red; position:relative; top:4px;">phonelink_ring</i></button>
      {/if}
      </div>
      

    </div>
    <p class="down-arrow">&darr;</p> 
    <div class="bag-container {isBagShaking ? 'shake' : ''}">
      <img src={bagImage} alt="Bag" class="bag" />
    </div>
    <!-- <div>
      <button>Charge Bag</button>
      <button>Unplug Charger</button>
    </div> -->
    
    

    <div class="phone {isShaking ? 'shake' : ''}">
      <div class="screen-content">
       
      
       <div class="screens">
       {#if (currentScreen==="home"|| "location" ||"batteryStatus"|| "lockStatus"||"customization"||"alerts"||"itemsLocation") && connection === false}
       <p class="heading">Smart Bag App</p>
       <button class="normalButton1 margin" on:click={connectBag}>Connect</button><br>
       
       {:else if currentScreen === 'screen1' && connection === true}
       <div class="screen1" >
        <p class="heading">Smart Bag App</p>
        <p style="margin-bottom:0px;">Connected to bag 01   <button class="normalButton1" on:click={disconnect}>Disconnect</button></p>
        <div style="display:flex; margin-top:0px;">
          <p class="paragraph" style="font-weight:bold;">Items in Bag</p>
        <p style="margin-left:20px; font-weight:bold; margin-bottom:0px;">Add Priority</p>
        </div>
        
        {#if items.length===0}
        <p>Bag Empty</p>
        {:else}
        <div class="items-container">
          <ul>
            {#each items as item}
              <li>
                <label class="checkbox-label">
                  {item}
                  <input 
                    type="checkbox" 
                    class="check-box"
                    checked={checkedItems.has(item)} 
                    on:change={() => toggleItem(item)} 
                  />
                  
                </label>
              </li>
            {/each}
          </ul>
        </div>
        {/if}
        <div>
          <p class="paragraph" style="font-weight:bold;">
            <i class="fas fa-exclamation-circle" style="color:red; font-size:15px;margin-top:0px;"></i> 
            Priority Items
            <button class="normalButton1" on:click={customize}>Add</button>
          </p>
          <div class="scrollable-important-items">
            <ul>
              {#each checkedItems as item}
              <li style="margin-left:0px;">
              {#if items.includes(item)}
              <span>
                <i class="fas fa-exclamation-circle" style="color:green; font-size:15px;"></i> 
              </span>
              {:else}
              <span>
                <i class="fas fa-exclamation-circle" style="color:red; font-size:15px;"></i> 
              </span>
              {/if}
              {item}
            </li>
              {/each}
            </ul>
          </div>
        </div>
       </div>

      {:else if currentScreen ==='location' && connection === true}
      
      <p class="heading"> Bag Location</p>
      <div class="bag-location">
       <BagLoc></BagLoc>
       <button style="margin-left:10px;"class="normalButton1" on:click={() => triggerBag(true)}>Ring my Bag</button>
       <button style="margin-left:10px;"class="normalButton1" on:click={items_location}>Find my items</button>
       
      </div>
      {:else if currentScreen ==="itemsLocation" && connection=== true}
       <div class="items-location">
        <p>My items are located at:</p>
        {#each Object.keys(items_loc) as i} 
        <p ><i style="font-size:15px;"class="fa fa-map-marker-alt"></i>{` ${i} : ${items_loc[i]} `}</p>
       {/each}
       </div> 
      {:else if currentScreen ==='battery' && connection === true}
      <div class="battery-status">
        <p class="heading">Battery Status</p>
        {#if batteryLevel<=20 && isCharging === false}
        <p><i class="fas fa-exclamation-triangle" style="color:red"></i>  Please Charge Your Battery!!!</p>
        {/if}
        <p>Battery status: {batteryLevel} %</p>
        {#if isCharging}
        <p><i class="fas fa-plug"></i>  Battery Charging</p>
        {/if}
        {#if remove_charger}
        <p><i class="fas fa-exclamation-triangle" style="color:red;"></i>   Please unplug the charger</p>
        {/if}
      </div>
      {:else if currentScreen ==="lockStatus" && connection === true}
      <div class="lock-status">
      <p class="heading">Lock Status</p>
      {#if lock}
      <p>Bag is locked <i class="fas fa-lock"></i></p>
      <button class="normalButton1" on:click={unlockBag}>Unlock</button>
      {:else}
      <p>Bag is Unlocked <i class="fas fa-unlock"></i></p>
      <button class ="normalButton1" on:click={lockBag}>Lock</button>
      {/if}
      </div>
      {:else if currentScreen ==="customization" && connection === true}
      <div class="criteria1">
        <p class="heading1">Set Important Items</p>
        <div class="criteria">
          <p>Select criteria</p>
          <select bind:value={criteria}>
            <option value="" disabled selected>Select a criterion</option>
            {#each Object.keys(criteriaItems) as criterion}
              <option value={criterion}>{criterion}</option>
            {/each}
          </select>
      
          {#if selectedItems.length > 0}
          <ul>
            {#each selectedItems as item}
              <li>
                <label>
                  <input 
                    type="checkbox" 
                    checked={checkedItems.has(item)} 
                    on:change={() => toggleItem(item)} 
                  />
                  {item}
                </label>
              </li>
            {/each}
          </ul>
          {/if}
        </div>
      </div>
      {:else if currentScreen ==="alerts" && connection ===true}
      <div class="notifications">
        {#if alerts.size===0}
      <p class="heading">No notifications</p>
      {:else}
      <p class="heading">Notifications</p>
      {#each alerts as a}
      <div class="alert">
        {a}
        <button class="close-btn" on:click={() => closeAlert(a)}>&times;</button>
      </div>
      {/each}
      {/if}
      
      </div>
      {/if}
      <!-- {#if batteryShow ==="no" && currentScreen==='home'} 
      <p class="heading2">Not Connected to any Smart Bag  &#10060;</p>

      {/if}
       -->
      <div class="phone-buttons">
        <nav>
          {#if connection === true}
          <div class="tooltip-container">
            <button class="nav-btn" on:click={homeStat}>
              <i class="fas fa-home"></i>
            </button>
            <div class="tooltip">Home</div>
          </div>
          {/if}
          <div class="tooltip-container">
          
            <button class="nav-btn" on:click={batteryStat}>
              <i class="fas fa-battery-full"></i>
            </button>
            <div class="tooltip">Battery Status</div>
          </div>
      
          <div class="tooltip-container">
            <button class="nav-btn" on:click={locSet}>
              <i class="fa fa-map-marker-alt"></i>
            </button>
            <div class="tooltip">Bag location</div>
          </div>
      
          <div class="tooltip-container">
            <button class="nav-btn" on:click={lockStat}>
              <i class="fas fa-lock"></i>
            </button>
            <div class="tooltip">Lock Status</div>
          </div>
      
          <div class="tooltip-container">
            <button class="nav-btn" on:click={notifications}>
              <i class="fas fa-bell"></i>
            </button>
            <div class="tooltip">Alerts</div>
            {#if alerts.size > 0 && connection===true}
            <!-- Show the badge if there are alerts -->
            <span class="notification-badge">{alerts.size}</span>
          {/if}
          </div>
        </nav>
      </div>
      
      </div>
      </div>
    </div>

    <div class="test-buttons">
      <p style="font-size:30px;font-weight:bold;margin-top:10px;">User Controls</p>
      <button class="normalButton" on:click={toggle}>Add item</button>
      {#if showInput}
      <br>
      <input type="text" bind:value={newItem} placeholder="Enter item name" />
      <button on:click={add_item} class="add-icon">
        Add
      </button>
    {/if}
    <br>
      <button class="normalButton" on:click={toggle_remove}>Remove item</button>
      {#if showInput1}
      <br>
      <input type="text" bind:value={newItem1} placeholder="Enter item name" />
      <button on:click={remove_item} class="remove-icon">
        Remove
      </button>
    {/if}
    <br>
    <button class="normalButton" on:click={() => triggerBag(false)} on:click={stuck_zipper}>Stuck Zipper</button><br>
    <button  on:click={chargeBattery} class="normalButton">Charge Bag</button><br>
    <button on:click={unPlugBattery} class="normalButton">Unplug Charger</button>
    <div class="test-ui">
      {#if addItem}
      <p class="test-notes">{addItemNotes}</p>
      {/if}
  
    </div>
      </div>
  </div>
  
  
    
  
</main>

<style>
  .checkbox-label {
  display: flex; /* Use flexbox to align items */
  justify-content: space-between; /* Space between text and checkbox */
  align-items: center; /* Center vertically */
}
.checkbox-label input{
  margin-left: 100px;
}
.down-arrow{
  font-size: 120px;
  position: relative;
  right: 21%;
  bottom:50px;
}




  
.items-location i {
  margin-right: 5px;  /* Space between the icon and the text */
  vertical-align: middle; /* Align icon vertically */
}
.icon-button{
  background-color: transparent;
  border:none;
  color:white;
  margin-left: 20px;
  margin-right: 10px;
}

.icon-overlay {
    position: relative;
    display: inline-block;
  }

  
    h1 {
    font-family: Arial, Helvetica, sans-serif;
    line-height: 1.5;
    font-weight: bold;
    text-align: center;
  }
  .screen1{
    margin-top: 150px;

  }
  
  .team-members{
    padding-left: 20px;
  }
  .paragraph{
    margin-bottom:0px;
  }
  .battery-status{
    margin-top: 200px;
  }
  .alert {
    padding: 10px;
    margin: 10px;
    margin-top: 30px;
    background-color: #f44336;
    color: white;
    border-radius: 4px;
    position: relative;
  }


  .close-btn {
    position: absolute;
    bottom:80%;
    left:90%;
    font-size: 20px;
   /* Adding a border to the close button */
    color:black;
    height:30px;
    padding-bottom: 2px;
  }

  .notification-badge {
    position: absolute;
    top: -10px;
    right: -10px;
    background-color: red;
    color: white;
    border-radius: 50%;
    padding: 5px 8px;
    font-size: 12px;
  }
  .items-container {
  max-height: 105px; /* Set the max height to restrict the list's size */
  overflow-y: auto;  /* Add vertical scroll when content overflows */
  margin-bottom: 20px;
  margin-right: 0px; /* Space below the scrollable list */
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
  padding-top: 0px;
  
  

  }
  .items-container::-webkit-scrollbar {
  width: 6px; /* Set width of the scrollbar */
  
  
}

.items-container::-webkit-scrollbar-thumb {
  background-color: white; /* Color of the scrollbar thumb */
  border-radius: 10px; /* Optional: Round corners */
}

.items-container::-webkit-scrollbar-track {
  background: transparent; /* Optional: Set the track background */
}

/* Hide arrows for vertical scrollbar */
.items-container::-webkit-scrollbar-button {
  display: none; /* This will remove the top and bottom arrows */
}

.scrollable-important-items {
  max-height: 90px; /* Set the max height to restrict the list's size */
  overflow-y: auto;  /* Add vertical scroll when content overflows */
  margin-bottom: 20px;
  margin-right: 0px; /* Space below the scrollable list */
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
  
  padding-top: 0px;
  
}
.scrollable-important-items::-webkit-scrollbar {
  width: 6px; /* Set width of the scrollbar */

}

.scrollable-important-items::-webkit-scrollbar-thumb {
  background-color: white; /* Color of the scrollbar thumb */
  border-radius: 10px; /* Optional: Round corners */
  opacity: 0.3;
}

.scrollable-important-items::-webkit-scrollbar-track {
  background: transparent; /* Optional: Set the track background */
}

/* Hide arrows for vertical scrollbar */
.scrollable-important-items::-webkit-scrollbar-button {
  display: none; /* This will remove the top and bottom arrows */
}
.items-location{
    margin-left: 60px;
    margin-top:40px;
    overflow-y: auto;
    overflow-x: hidden;
    height:400px;
    margin-right:30px;
    

}
.items-location::-webkit-scrollbar {
  width: 6px; /* Set width of the scrollbar */

}

.items-location::-webkit-scrollbar-thumb {
  background-color:rgba(0, 0, 0, 0.3); /* Color of the scrollbar thumb */
  border-radius: 10px; /* Optional: Round corners */
  
  

}

.items-location::-webkit-scrollbar-track {
  background: transparent; /* Optional: Set the track background */
}

/* Hide arrows for vertical scrollbar */
.items-location::-webkit-scrollbar-button {
  display: none; /* This will remove the top and bottom arrows */
}
.notifications{
  overflow-y: auto;
  overflow-x: hidden;
  height:300px;
  margin-top: 100px;
}
.notifications::-webkit-scrollbar {
  width: 6px; /* Set width of the scrollbar */

}

.notifications::-webkit-scrollbar-thumb {
  background-color:rgba(185, 181, 181, 0.5); /* Color of the scrollbar thumb */
  border-radius: 10px; /* Optional: Round corners */
  
}

.notifications::-webkit-scrollbar-track {
  background: transparent; /* Optional: Set the track background */
}

/* Hide arrows for vertical scrollbar */
.notifications::-webkit-scrollbar-button {
  display: none; /* This will remove the top and bottom arrows */
}

.bag-location{
  overflow-y: auto;
  overflow-x: auto;
  height:420px;
  margin-top: 50px;
  max-width: 300px;
}
.bag-location::-webkit-scrollbar {
  width: 6px; /* Set width of the scrollbar */

}

.bag-location::-webkit-scrollbar-thumb {
  background-color:rgba(185, 181, 181, 0.5); /* Color of the scrollbar thumb */
  border-radius: 10px; /* Optional: Round corners */
  
}

.bag-location::-webkit-scrollbar-track {
  background: transparent; /* Optional: Set the track background */
}

/* Hide arrows for vertical scrollbar */
.bag-location::-webkit-scrollbar-button {
  display: none; /* This will remove the top and bottom arrows */
}


ul {
  list-style: none; /* Remove default bullet points */
  padding: 0;
  margin: 0;
}

li {
  margin: 5px 0; /* Space between list items */
}

  
.lock-status{
  margin-top:200px;
}
  
  .margin{
    margin-top:300px;
  }

  nav {
  position: absolute; /* Fixed within the phone container */
  width: 100%; /* Full width of the phone screen */
  display: flex;
  justify-content: space-around;
  padding: 10px;
  bottom: 0; /* Align it to the bottom of the phone screen */
  left: 0; /* Align to the left */
  background-color:transparent; /* Optional: Semi-transparent background */
  border-top-left-radius: 20px;
  border-top-right-radius: 20px;
  z-index: 1000; /* Ensure it stays above content */
}

.screen-content {
  width: 300px;
  height: 520px;
  background-color: black;
  color: white;
  border-radius: 35px; /* Internal rounded screen */
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
   /* Enable vertical scrolling */
  padding-bottom: 60px; /* Leave space for the nav bar */
  position: relative; /* Ensure the nav bar stays inside the phone */
}

  .nav-btn {
    color: white;
    background-color: black;
    border: none;
    font-size: 24px;
    cursor: pointer;
  }

  .nav-btn:hover {
    color: #aaa;
  }
  .nav-btn[disabled] {
    opacity: 0.5; /* Make the button appear faded */
    cursor: not-allowed; /* Change the cursor to indicate it's disabled */
  }
  
 
  button:hover {
    background-color: gray;
  }
  
  .doc {
    text-decoration: none;
    color: white;
  }
  
  .container {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    margin-top: 20px; /* Spacing above the container */
  }
  i{
    font-size: 24px;
  }
  
   .bag-container {
    position: relative;
    width: 300px;
    height: auto;
    right:30%;
    margin-top:200px;
  } 
  
  .bag {
    width: 100%; /* Automatically fits within the bag-container */
    height: auto;
  
  }

  .bag-screen {
    width: 300px;
    height: 50px;
    border: 10px solid black;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background-color: black;
    margin-left:20px;
    margin-top:10px;
  }

  .phone {
    width: 320px;
    height: 540px;
    border-radius: 40px;
    border: 3px solid rgba(0, 0, 0, 0.3); /* Thin bezel */
    display: flex;
    align-items: center;
    justify-content: center;
    background: linear-gradient(145deg, #e0e0e0, #ffffff);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15); /* Subtle shadow for depth */
    margin-right: 100px; /* Space between phone and bag */
    position: relative;
    overflow: hidden;
    right:15%;
   /* Space between phone and bag */
  }

 

  /* Button container inside phone screen */
  .phone-buttons {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-around;
    width: 100%;
    padding: 10px;
    margin-top:30px;
  }
  .heading {
    font-size: 25px;
    font-weight: bold;
    position: absolute; /* Make sure it is positioned relative to the container */
    top: 5%; /* Adjust this percentage to control how close it is to the top */
    left: 50%; /* Center the heading horizontally */
    transform: translateX(-50%); /* Align it to the center */
    text-align: center;
    color: white; /* Adjust if necessary for visibility */
  }
  .heading1{
    font-size: 20px;
    font-weight: bold;
    position: absolute; /* Make sure it is positioned relative to the container */
    top: 5%; /* Adjust this percentage to control how close it is to the top */
    left: 50%; /* Center the heading horizontally */
    transform: translateX(-50%); /* Align it to the center */
    text-align: center;
    color: white;
  }

.test-ui{
    width:200px;
    height:100px;
    border: 1px solid black;
    position:relative;
   
}

/* Shake animation */
@keyframes shake {
    0%, 100% {
      transform: translateX(0);
    }
    25% {
      transform: translateX(-10px);
    }
    50% {
      transform: translateX(10px);
    }
    75% {
      transform: translateX(-10px);
    }
  }

  .shake {
    animation: shake 0.5s ease-in-out infinite; /* Shake effect lasts continuously */
  }

  .tooltip-container {
  position: relative;
  display: inline-block;
}


.tooltip {
  position: absolute;
  background-color: #333;
  color: #fff;
  padding: 8px;
  border-radius: 5px;
  font-size: 12px;
  white-space: nowrap;
  z-index: 1;
  top: -40px;
  left: 50%;
  transform: translateX(-50%);
  opacity: 0;
  transition: opacity 0.3s ease;
  pointer-events: none; /* Prevent interaction with tooltip */
}

/* Show tooltip on hover */
.tooltip-container:hover .tooltip {
  opacity: 1;
}
.test-buttons {
  gap: 10px; /* Add space between the buttons */
  justify-content: center; /* Center the buttons horizontally */
  margin-bottom: 20px; /* Space between buttons and the next content */
  /* Remove transform to avoid shifting */
  position: relative;
  right:10%;
}
.test-notes{
  padding:10px;
}

.criteria{
  margin-top:120px;
  overflow-y: auto;
  max-height: 500px;
  margin-right: 80px;
}


</style>
