---
sidebar: auto
---

# Config

## Corona widget

````
// Variables used by Scriptable.
// These must be at the very top of the file. Do not edit.
// icon-color: deep-green; icon-glyph: user-md;
// change "country" to a value from https://coronavirus-19-api.herokuapp.com/countries/
const country = "France"
const url = `https://coronavirus-19-api.herokuapp.com/countries/${country}`
const req = new Request(url)
const res = await req.loadJSON()

if (config.runsInWidget) {
  // create and show widget
  let widget = createWidget("Coronavirus", `${res.todayCases} Today`, `${res.cases} Total`, "#53d769")
  Script.setWidget(widget)
  Script.complete()
} else {
  // make table
  let table = new UITable()
  
  // add header
  let row = new UITableRow()
  row.isHeader = true
  row.addText(`Coronavirus Stats in ${country}`)
  table.addRow(row)
  
  // fill data
  table.addRow(createRow("Cases", res.cases))
  table.addRow(createRow("Today", res.todayCases))
  table.addRow(createRow("Deaths", res.deaths))
  table.addRow(createRow("Recovered", res.recovered))
  table.addRow(createRow("Critical", res.critical))
  
  if (config.runsWithSiri)
    Speech.speak(`There are ${res.cases} cases in ${country}, and ${res.todayCases} cases today.`)
  
  // present table
  table.present()
}

function createRow(title, number) {
  let row = new UITableRow()
  row.addText(title)
  row.addText(number.toString()).rightAligned()
  return row
}

function createWidget(pretitle, title, subtitle, color) {
  let w = new ListWidget()
  w.backgroundColor = new Color(color)
  let preTxt = w.addText(pretitle)
  preTxt.textColor = Color.white()
  preTxt.textOpacity = 0.8
  preTxt.font = Font.systemFont(16)
  w.addSpacer(5)
  let titleTxt = w.addText(title)
  titleTxt.textColor = Color.white()
  titleTxt.font = Font.systemFont(22)
  w.addSpacer(5)
  let subTxt = w.addText(subtitle)
  subTxt.textColor = Color.white()
  subTxt.textOpacity = 0.8
  subTxt.font = Font.systemFont(18)
  return w
}

````

## top_500_albums_widget

````
// insert your Spotify client id and secret here
let clientId = "xxx"
let clientSecret = "xxx"

// use your spotify country iso code to optimize search results
let spotifyCountry = "DE"

// optional: the ip of your node-sonos-http-api and room name; use "sonos" as parameter in your widget settings to activate it
let sonosUrl = "http://192.168.178.10:5005/Kitchen"

let openWith = args.widgetParameter
let widget = new ListWidget()
widget.setPadding(0,0,0,0)

widget.backgroundColor = new Color("#000000")
let searchToken = await getSpotifySearchToken()
await getRandomAlbum()

Script.setWidget(widget)
Script.complete()

async function getRandomAlbum() {
  // load json from iCloud Drive
  // source: https://gist.github.com/marco79cgn/92092f4a4f05434efe84f7191e55a8de
  let fm = FileManager.iCloud()
  let dir = fm.documentsDirectory()
  let path = fm.joinPath(dir, "top_500_albums.json")
  let contents = Data.fromFile(path)
  let jsonTop500 = JSON.parse(contents.toRawString())
  let uri = ""
  let externalUrl = ""
  let coverUrl = ""
  
  do {
    let randomNumber = getRandomNumber(1, 500)
    let result = await searchAlbumAtSpotify(jsonTop500[randomNumber-1].Album, jsonTop500[randomNumber-1].Artist)
    // verify spotify result and query next album if empty/not available
    if (result != null && result.albums != null && result.albums.items != null && result.albums.items.length == 1) {
      let item = result.albums.items[0]
      coverUrl = item.images[0].url
      uri = item.uri
      externalUrl = item.external_urls.spotify
    }
  } while(coverUrl.length == 0)

  if(openWith != null && openWith === "sonos") {
    widget.url = sonosUrl + "/spotify/now/" + uri
  } else {
    widget.url = externalUrl
  }
  await loadImage(coverUrl)
}

// gets a spotify search token
async function getSpotifySearchToken() {
  let url = "https://accounts.spotify.com/api/token"
  let req = new Request(url)
  req.method = "POST"
  req.body = "grant_type=client_credentials"
  let authHeader = "Basic " + btoa(clientId+":"+clientSecret)
  req.headers = {"Authorization": authHeader, "Content-Type":"application/x-www-form-urlencoded"}
  let token = await req.loadJSON()
  return token.access_token
}

// random number, min and max included 
function getRandomNumber(min, max) {
    return Math.floor(Math.random() * (max - min + 1) + min)
}

// search for the album at Spotify
async function searchAlbumAtSpotify(album, artist) {
  let searchString = encodeURIComponent("album:" + album +" artist:" + artist)
  let searchUrl = "https://api.spotify.com/v1/search?q=" + searchString + "&type=album&market=" + spotifyCountry + "&limit=1"
  req = new Request(searchUrl)
  req.headers = {"Authorization": "Bearer " + searchToken, "Content-Type":"application/json", "Accept":"application/json"}
  let result = await req.loadJSON()
  return result
}

// download and display the cover
async function loadImage(imageUrl) {
  let req = new Request(imageUrl)
  let image = await req.loadImage()
  let widgetImage = widget.addImage(image)
  widgetImage.imageSize = new Size(148,148)
  widgetImage.centerAlignImage()
  widgetImage.cornerRadius = 10
}

````

## Affichage de la date transparent

````
// insert your Spotify client id and secret here
let clientId = "xxx"
let clientSecret = "xxx"

// use your spotify country iso code to optimize search results
let spotifyCountry = "DE"

// optional: the ip of your node-sonos-http-api and room name; use "sonos" as parameter in your widget settings to activate it
let sonosUrl = "http://192.168.178.10:5005/Kitchen"

let openWith = args.widgetParameter
let widget = new ListWidget()
widget.setPadding(0,0,0,0)

widget.backgroundColor = new Color("#000000")
let searchToken = await getSpotifySearchToken()
await getRandomAlbum()

Script.setWidget(widget)
Script.complete()

async function getRandomAlbum() {
  // load json from iCloud Drive
  // source: https://gist.github.com/marco79cgn/92092f4a4f05434efe84f7191e55a8de
  let fm = FileManager.iCloud()
  let dir = fm.documentsDirectory()
  let path = fm.joinPath(dir, "top_500_albums.json")
  let contents = Data.fromFile(path)
  let jsonTop500 = JSON.parse(contents.toRawString())
  let uri = ""
  let externalUrl = ""
  let coverUrl = ""
  
  do {
    let randomNumber = getRandomNumber(1, 500)
    let result = await searchAlbumAtSpotify(jsonTop500[randomNumber-1].Album, jsonTop500[randomNumber-1].Artist)
    // verify spotify result and query next album if empty/not available
    if (result != null && result.albums != null && result.albums.items != null && result.albums.items.length == 1) {
      let item = result.albums.items[0]
      coverUrl = item.images[0].url
      uri = item.uri
      externalUrl = item.external_urls.spotify
    }
  } while(coverUrl.length == 0)

  if(openWith != null && openWith === "sonos") {
    widget.url = sonosUrl + "/spotify/now/" + uri
  } else {
    widget.url = externalUrl
  }
  await loadImage(coverUrl)
}

// gets a spotify search token
async function getSpotifySearchToken() {
  let url = "https://accounts.spotify.com/api/token"
  let req = new Request(url)
  req.method = "POST"
  req.body = "grant_type=client_credentials"
  let authHeader = "Basic " + btoa(clientId+":"+clientSecret)
  req.headers = {"Authorization": authHeader, "Content-Type":"application/x-www-form-urlencoded"}
  let token = await req.loadJSON()
  return token.access_token
}

// random number, min and max included 
function getRandomNumber(min, max) {
    return Math.floor(Math.random() * (max - min + 1) + min)
}

// search for the album at Spotify
async function searchAlbumAtSpotify(album, artist) {
  let searchString = encodeURIComponent("album:" + album +" artist:" + artist)
  let searchUrl = "https://api.spotify.com/v1/search?q=" + searchString + "&type=album&market=" + spotifyCountry + "&limit=1"
  req = new Request(searchUrl)
  req.headers = {"Authorization": "Bearer " + searchToken, "Content-Type":"application/json", "Accept":"application/json"}
  let result = await req.loadJSON()
  return result
}

// download and display the cover
async function loadImage(imageUrl) {
  let req = new Request(imageUrl)
  let image = await req.loadImage()
  let widgetImage = widget.addImage(image)
  widgetImage.imageSize = new Size(148,148)
  widgetImage.centerAlignImage()
  widgetImage.cornerRadius = 10
}

````

## Unsplash random img

````
var keyword="geek"
if(args.widgetParameter){
  keyword=args.widgetParameter
}
const result=await get({url:"https://api.unsplash.com/search/photos?page=2&per_page=20&query="+keyword+"&collections&orientation=portrait",headers:{"Accept-Encoding": "gzip, deflate, br","Accept-Language": "zh-cn","Authorization": "Client-ID 9657b2982a53f8bf4b567fe7899da7354456296f0d91a2f918a1bbcfec8a021e","Connection": "keep-alive","Cookie": "ugid=12cbed22cf668302ec43f4b7536e66235337523","Host": "api.unsplash.com","User-Agent": "Unsplash/73 CFNetwork/1197 Darwin/20.0.0",}})

const total=result.results.length
const index=Math.floor(Math.random()*total) 
const imgUrl=result.results[index].urls.regular
const req = await new Request(imgUrl);
const img = await req.loadImage();
const w = new ListWidget()
w.backgroundImage=img
w.url=imgUrl
let widget = w
Script.setWidget(widget)
Script.complete()
w.presentMedium()
 
  async function get(opts) {
      const request = new Request(opts.url)
      request.headers = {
        ...opts.headers,
        ...this.defaultHeaders
      }
      var result=await request.loadJSON()
      console.log(result)
      return result
    
}
````
## Random Painting

````
//Fetches Random Images from the metmuseum-api ; Stores the ids in the keychain in order to prevent multiple calls
//Department ID = 11 is paintings ; use other if you wish
var keychainkey = "paintingIDs"
var res
if(!Keychain.contains(keychainkey)) {
  console.log("Keychainentry does not exist... create it");
  const url = 'https://collectionapi.metmuseum.org/public/collection/v1/search?hasImages=true&medium=Paintings&departmentId=11&q=Painting'
  const req = new Request(url)
  res = await req.loadJSON()
  let stringified = JSON.stringify(res)
  Keychain.set(keychainkey, stringified)
}
else {
  console.log("Using keychainentry now")
  var idsFromKeychain = Keychain.get(keychainkey)
//   console.log(idsFromKeychain)
  res = JSON.parse(idsFromKeychain)
//   console.log(res)
}
const max = res.total-1
console.log("Max: "+max);
const min = 0
const random = Math.floor(Math.random() * (max - min + 1)) + min
console.log("Random: "+random);
const picked = res.objectIDs[random]
console.log("Picked: "+picked);
const req2 = new Request('https://collectionapi.metmuseum.org/public/collection/v1/objects/'+picked)
// const req2 = new Request("https://collectionapi.metmuseum.org/public/collection/v1/objects/436978")
const res2 = await req2.loadJSON()
const i = new Request(res2.primaryImageSmall);
const img = await i.loadImage();

let widget = createWidget(img, res2.objectURL)
if (config.runsInWidget) {
  // create and show widget
  Script.setWidget(widget)
  Script.complete()
}
else {
  widget.presentLarge()
}


function createWidget(img, widgeturl) {
  let w = new ListWidget()
  w.backgroundImage = img
  w.url = widgeturl
  return w
}

````

## invisible widget


````
// Variables used by Scriptable.
// These must be at the very top of the file. Do not edit.
// icon-color: deep-purple; icon-glyph: image;

// This widget was created by Max Zeryck @mzeryck

// Widgets are unique based on the name of the script.
const filename = Script.name() + ".jpg"
const files = FileManager.local()
const path = files.joinPath(files.documentsDirectory(), filename)

if (config.runsInWidget) {
  let widget = new ListWidget()
  widget.backgroundImage = files.readImage(path)
  
  // You can your own code here to add additional items to the "invisible" background of the widget.
  
  Script.setWidget(widget)
  Script.complete()

/*
 * The code below this comment is used to set up the invisible widget.
 * ===================================================================
 */
} else {
  
  // Determine if user has taken the screenshot.
  var message
  message = "Before you start, go to your home screen and enter wiggle mode. Scroll to the empty page on the far right and take a screenshot."
  let exitOptions = ["Continue","Exit to Take Screenshot"]
  let shouldExit = await generateAlert(message,exitOptions)
  if (shouldExit) return
  
  // Get screenshot and determine phone size.
  let img = await Photos.fromLibrary()
  let height = img.size.height
  let phone = phoneSizes()[height]
  if (!phone) {
    message = "It looks like you selected an image that isn't an iPhone screenshot, or your iPhone is not supported. Try again with a different image."
    await generateAlert(message,["OK"])
    return
  }
  
  // Prompt for widget size and position.
  message = "What size of widget are you creating?"
  let sizes = ["Small","Medium","Large"]
  let size = await generateAlert(message,sizes)
  let widgetSize = sizes[size]
  
  message = "What position will it be in?"
  message += (height == 1136 ? " (Note that your device only supports two rows of widgets, so the middle and bottom options are the same.)" : "")
  
  // Determine image crop based on phone size.
  let crop = { w: "", h: "", x: "", y: "" }
  if (widgetSize == "Small") {
    crop.w = phone.small
    crop.h = phone.small
    let positions = ["Top left","Top right","Middle left","Middle right","Bottom left","Bottom right"]
    let position = await generateAlert(message,positions)
    
    // Convert the two words into two keys for the phone size dictionary.
    let keys = positions[position].toLowerCase().split(' ')
    crop.y = phone[keys[0]]
    crop.x = phone[keys[1]]
    
  } else if (widgetSize == "Medium") {
    crop.w = phone.medium
    crop.h = phone.small
    
    // Medium and large widgets have a fixed x-value.
    crop.x = phone.left
    let positions = ["Top","Middle","Bottom"]
    let position = await generateAlert(message,positions)
    let key = positions[position].toLowerCase()
    crop.y = phone[key]
    
  } else if(widgetSize == "Large") {
    crop.w = phone.medium
    crop.h = phone.large
    crop.x = phone.left
    let positions = ["Top","Bottom"]
    let position = await generateAlert(message,positions)
    
    // Large widgets at the bottom have the "middle" y-value.
    crop.y = position ? phone.middle : phone.top
  }
  
  // Crop image and finalize the widget.
  let imgCrop = cropImage(img, new Rect(crop.x,crop.y,crop.w,crop.h))
  
  message = "Your widget background is ready. Would you like to use it in a Scriptable widget or export the image?"
  const exportPhotoOptions = ["Use in Scriptable","Export to Photos"]
  const exportPhoto = await generateAlert(message,exportPhotoOptions)
  
  if (exportPhoto) {
    Photos.save(imgCrop)
  } else {
    files.writeImage(path,imgCrop)
  }
  
  Script.complete()
}

// Generate an alert with the provided array of options.
async function generateAlert(message,options) {
  
  let alert = new Alert()
  alert.message = message
  
  for (const option of options) {
    alert.addAction(option)
  }
  
  let response = await alert.presentAlert()
  return response
}

// Crop an image into the specified rect.
function cropImage(img,rect) {
   
  let draw = new DrawContext()
  draw.size = new Size(rect.width, rect.height)
  
  draw.drawImageAtPoint(img,new Point(-rect.x, -rect.y))  
  return draw.getImage()
}

// Pixel sizes and positions for widgets on all supported phones.
function phoneSizes() {
  let phones = {	
	"2688": {
			"small":  507,
			"medium": 1080,
			"large":  1137,
			"left":  81,
			"right": 654,
			"top":    228,
			"middle": 858,
			"bottom": 1488
	},
	
	"1792": {
			"small":  338,
			"medium": 720,
			"large":  758,
			"left":  54,
			"right": 436,
			"top":    160,
			"middle": 580,
			"bottom": 1000
	},
	
	"2436": {
			"small":  465,
			"medium": 987,
			"large":  1035,
			"left":  69,
			"right": 591,
			"top":    213,
			"middle": 783,
			"bottom": 1353
	},
	
	"2208": {
			"small":  471,
			"medium": 1044,
			"large":  1071,
			"left":  99,
			"right": 672,
			"top":    114,
			"middle": 696,
			"bottom": 1278
	},
	
	"1334": {
			"small":  296,
			"medium": 642,
			"large":  648,
			"left":  54,
			"right": 400,
			"top":    60,
			"middle": 412,
			"bottom": 764
	},
	
	"1136": {
			"small":  282,
			"medium": 584,
			"large":  622,
			"left": 30,
			"right": 332,
			"top":  59,
			"middle": 399,
			"bottom": 399
	}
  }
  return phones
=======
// icon-color: gray; icon-glyph: angle-right;

// script     : xkcdWidget.js
// version    : 1.0.0 
// description: xkcd widget for Scriptable.app
// author     : @supermamon
// date       : 2020-09-17


const BACKGROUND_DARK_MODE = "system" 
// options: "yes", "no", "system"

let RANDOM = false
// default is current comic
// set the Parameter value to "random" in the 
// Edit Widget screen to use a random comic
if (args.widgetParameter == 'random') {
  RANDOM = true
}

// show the alt text at the bottom of the image.
let SHOW_ALT = true

// load data and create widget
let data = await loadData()
let widget = await createWidget(data)

if (!config.runsInWidget) {
     await widget.presentLarge()
}

// Tell the system to show the widget.
Script.setWidget(widget) 
Script.complete()

async function createWidget(data) {
  const w = new ListWidget();

  let isDarkMode = 
    BACKGROUND_DARK_MODE=="system" ? 
    await isUsingDarkAppearance() : 
    BACKGROUND_DARK_MODE=="yes"

  if (isDarkMode) {
    w.backgroundGradient = newLinearGradient('#010c1ee6','#001e38b3')
  } else {
    w.backgroundGradient = newLinearGradient('#b00a0fe6','#b00a0fb3')
  }
  
  let titleTxt = w.addText(data.safe_title)
  titleTxt.font = Font.boldSystemFont(14)
  titleTxt.centerAlignText()
  titleTxt.textColor = Color.white()

  w.addSpacer(2)

  let img = await downloadImage(data.img);
  let pic = w.addImage(img)
  pic.centerAlignImage()

  w.addSpacer()

  if (SHOW_ALT) {
    let subTxt = w.addText(`${data.num}: ${data.alt}`)
    subTxt.font = Font.mediumSystemFont(10)
    subTxt.textColor = Color.white()
    subTxt.textOpacity = 0.9
    subTxt.centerAlignText()
  }

  return w
}
async function loadData() {
  return (await xkcd(RANDOM))
}
function newLinearGradient(from, to) {
  let gradient = new LinearGradient()
  gradient.locations = [0, 1]
  gradient.colors = [
    new Color(from),
    new Color(to)
  ]
  return gradient
}
async function downloadImage(imgurl) {
  let imgReq = new Request(imgurl)
  let img = await imgReq.loadImage()
  return img
}
async function xkcd(random) {
  let url = "https://xkcd.com/info.0.json"
  let req = new Request(url)
  let json = await req.loadJSON()

  if (random) {
    let rnd = Math.floor(Math.random() * (json.num - 1 + 1) ) + 1
    url = `https://xkcd.com/${rnd}/info.0.json`
    req = new Request(url)
    json = await req.loadJSON()
  } 

  return json
}

async function isUsingDarkAppearance() {
  // yes there's a Device.isUsingDarkAppearance() method
  // but I find it unreliable
  const wv = new WebView()
  let js ="(window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches)"
  let r = await wv.evaluateJavaScript(js)
  return r
}

````
## Frankenstein widget

READ THE INSTRUCTIONS BELOW
You need to both edit the code and add parameters in order to personalise the widget.
To add parameters, add the widget to your home, long press it and tap 'edit widget'
Your parameter must have the following format: image.png|padding-top|text-color|
The image should be placed in the iCloud Scriptable folder (case-sensitive).
The padding-top spacing parameter moves the text down by a set amount.
The text color parameter should be a hex value.
For example, to use the image bkg_fall.PNG with a padding of 20 and a text color of red
the parameter should be typed as: bkg_fall.png|20|#ff0000(rouge) #384b77(bleu) #FFFFFF(blanc) #0000ff(bleu style phantom)
All parameters are required and separated with "|"
Parameters allow different settings for multiple widget instances.

````
// Variables used by Scriptable.
// These must be at the very top of the file. Do not edit.
// icon-color: blue; icon-glyph: calendar-alt;

// CREDITS
// Greetings code Autumn Vibes by u/ben5292001
// Battery code by u/_Bisho_
// Weather stack code by u/coderjones
// Countdown code and assemble by u/flasozzi

// READ THE INSTRUCTIONS BELOW
// You need to both edit the code and add parameters in order to personalise the widget.

// To add parameters, add the widget to your home, long press it and tap 'edit widget'
// Your parameter must have the following format: image.png|padding-top|text-color|
// The image should be placed in the iCloud Scriptable folder (case-sensitive).
// The padding-top spacing parameter moves the text down by a set amount.
// The text color parameter should be a hex value.
// For example, to use the image bkg_fall.PNG with a padding of 20 and a text color of red
// the parameter should be typed as: bkg_fall.png|20|#ff0000
// All parameters are required and separated with "|"
// Parameters allow different settings for multiple widget instances.

// You also need to edit the info below with your own info.
// You can grab your weather API and city ID from openweathermap.org (free account needed).

var yourAPI = "40b532eb499b6315640892e324f7efce";
var cityID = "2996943";
var userName = "Emma";
var countdownTo = "December 25 2020";
var countdownTitle = "Noeeel";


// Now this is where the fun begins and you don't really need to do anything else from this point on.


let widgetHello = new ListWidget(); 
var today = new Date();

var widgetInputRAW = args.widgetParameter;

try {
	widgetInputRAW.toString();
} catch(e) {
	throw new Error("Please long press the widget and add a parameter.");
}

var widgetInput = widgetInputRAW.toString();

var inputArr = widgetInput.split("|");

var spacing = parseInt(inputArr[1]);

// iCloud file path
var scriptableFilePath = "/var/mobile/Library/Mobile Documents/iCloud~dk~simonbs~Scriptable/Documents/";
var removeSpaces1 = inputArr[0].split(" "); // Remove spaces from file name
var removeSpaces2 = removeSpaces1.join('');
var tempPath = removeSpaces2.split(".");
var backgroundImageURLRAW = scriptableFilePath + tempPath[0];

var fm = FileManager.iCloud();
var backgroundImageURL = scriptableFilePath + tempPath[0] + ".";
var backgroundImageURLInput = scriptableFilePath + removeSpaces2;

// For users having trouble with extensions
// Uses user-input file path is the file is found
// Checks for common file format extensions if the file is not found
if (fm.fileExists(backgroundImageURLInput) == false) {
		var fileTypes = ['png', 'jpg', 'jpeg', 'tiff', 'webp', 'gif'];

		fileTypes.forEach(function(item) {
			if (fm.fileExists((backgroundImageURL + item.toLowerCase())) == true) {
				backgroundImageURL = backgroundImageURLRAW + "." + item.toLowerCase();
			} else if (fm.fileExists((backgroundImageURL + item.toUpperCase())) == true) {
				backgroundImageURL = backgroundImageURLRAW + "." + item.toUpperCase();
			}
		});
} else {
	backgroundImageURL = scriptableFilePath + removeSpaces2;
}

var spacing = parseInt(inputArr[1]);

//API_KEY
let API_WEATHER = yourAPI;
let CITY_WEATHER = cityID;

//Get storage
var base_path = "/var/mobile/Library/Mobile Documents/iCloud~dk~simonbs~Scriptable/Documents/weather/";
var fm = FileManager.iCloud();

// Fetch Image from Url
async function fetchimageurl(url) {
	const request = new Request(url)
	var res = await request.loadImage();
	return res;
}

// Get formatted Date
function getformatteddate(){
  var months = ['January','February','March','April','May','June','July','August','September','October','November','December'];
  return months[today.getMonth()] + " " + today.getDate()
}

// Long-form days and months
var days = ['Sunday','Monday','Tuesday','Wednesday','Thursday','Friday','Saturday'];
var months = ['January','February','March','April','May','June','July','August','September','October','November','December'];

// Load image from local drive
async function fetchimagelocal(path){
  var finalPath = base_path + path + ".png";
  if(fm.fileExists(finalPath)==true){
	console.log("file exists: " + finalPath);
	return finalPath;
  }else{
	//throw new Error("Error file not found: " + path);
	if(fm.fileExists(base_path)==false){
	  console.log("Directry not exist creating one.");
	  fm.createDirectory(base_path);
	}
	console.log("Downloading file: " + finalPath);
	await downloadimg(path);
	if(fm.fileExists(finalPath)==true){
	  console.log("file exists after download: " + finalPath);
	  return finalPath;
	}else{
	  throw new Error("Error file not found: " + path);
	}
  }
}

async function downloadimg(path){
	const url = "http://a.animedlweb.ga/weather/weathers25_2.json";
	const data = await fetchWeatherData(url);
	var dataimg = null;
	var name = null;
	if(path.includes("bg")){
	  dataimg = data.background;
	  name = path.replace("_bg","");
	}else{
	  dataimg = data.icon;
	  name = path.replace("_ico","");
	}
	var imgurl=null;
	switch (name){
	  case "01d":
		imgurl = dataimg._01d;
	  break;
	  case "01n":
		imgurl = dataimg._01n;
	  break;
	  case "02d":
		imgurl = dataimg._02d;
	  break;
	  case "02n":
		imgurl = dataimg._02n;
	  break;
	  case "03d":
		imgurl = dataimg._03d;
	  break;
	  case "03n":
		imgurl = dataimg._03n;
	  break;
	  case "04d":
		imgurl = dataimg._04d;
	  break;
	  case "04n":
		imgurl = dataimg._04n;
	  break;
	  case "09d":
		imgurl = dataimg._09d;
	  break;
	  case "09n":
		imgurl = dataimg._09n;
	  break;
	  case "10d":
		imgurl = dataimg._10d;
	  break;
	  case "10n":
		imgurl = dataimg._10n;
	  break;
	  case "11d":
		imgurl = dataimg._11d;
	  break;
	  case "11n":
		imgurl = dataimg._11n;
	  break;
	  case "13d":
		imgurl = dataimg._13d;
	  break;
	  case "13n":
		imgurl = dataimg._13n;
	  break;
	  case "50d":
		imgurl = dataimg._50d;
	  break;
	  case "50n":
		imgurl = dataimg._50n;
	  break;
	}
	const image = await fetchimageurl(imgurl);
	console.log("Downloaded Image");
	fm.writeImage(base_path+path+".png",image);
}

//get Json weather
async function fetchWeatherData(url) {
  const request = new Request(url);
  const res = await request.loadJSON();
  return res;
}

// Get Location 
/*Location.setAccuracyToBest();
let curLocation = await Location.current();
console.log(curLocation.latitude);
console.log(curLocation.longitude);*/
let wetherurl = "http://api.openweathermap.org/data/2.5/weather?id=" + CITY_WEATHER + "&APPID=" + API_WEATHER + "&units=metric";
//"http://api.openweathermap.org/data/2.5/weather?lat=" + curLocation.latitude + "&lon=" + curLocation.longitude + "&appid=" + API_WEATHER + "&units=metric";
//"http://api.openweathermap.org/data/2.5/weather?id=" + CITY_WEATHER + "&APPID=" + API_WEATHER + "&units=metric"

const weatherJSON = await fetchWeatherData(wetherurl);
const cityName = weatherJSON.name;
const weatherarry = weatherJSON.weather;
const iconData = weatherarry[0].icon;
const weathername = weatherarry[0].main;
const curTempObj = weatherJSON.main;
const curTemp = curTempObj.temp;
const highTemp = curTempObj.temp_max;
const lowTemp = curTempObj.temp_min;
const feel_like = curTempObj.feels_like;
//Completed loading weather data

// Greetings arrays per time period. 
var greetingsMorning = [
'Good morning, ' + userName + '.'
];
var greetingsAfternoon = [
'Good afternoon, ' + userName + '.'
];
var greetingsEvening = [
'Good evening, ' + userName + '.'
];
var greetingsNight = [
'Good night, ' + userName + '.'
];
var greetingsLateNight = [
'Go to sleep, ' + userName + '.'
];

// Holiday customization
var holidaysByKey = {
	// month,week,day: datetext
	"11,4,4": "Happy Thanksgiving!"
}

var holidaysByDate = {
	// month,date: greeting
	"1,1": "Happy " + (today.getFullYear()).toString() + "!",
	"10,31": "Happy Halloween!",
	"12,25": "Merry Christmas!"
}

var holidayKey = (today.getMonth() + 1).toString() + "," +  (Math.ceil(today.getDate() / 7)).toString() + "," + (today.getDay()).toString();

var holidayKeyDate = (today.getMonth() + 1).toString() + "," + (today.getDate()).toString();

// Date Calculations
var weekday = days[ today.getDay() ];
var month = months[ today.getMonth() ];
var date = today.getDate();
var hour = today.getHours();

// Append ordinal suffix to date
function ordinalSuffix(input) {
	if (input % 10 == 1 && date != 11) {
		return input.toString() + "st";
	} else if (input % 10 == 2 && date != 12) {
		return input.toString() + "nd";
	} else if (input % 10 == 3 && date != 13) {
		return input.toString() + "rd";
	} else {
		return input.toString() + "th";
	}
}

// Generate date string
var datefull = weekday + ", " + month + " " + ordinalSuffix(date);

// Support for multiple greetings per time period
function randomGreeting(greetingArray) {
	return Math.floor(Math.random() * greetingArray.length);
}

var greeting = new String("Howdy.")
if (hour < 5 && hour >= 1) { // 1am - 5am
	greeting = greetingsLateNight[randomGreeting(greetingsLateNight)];
} else if (hour >= 23 || hour < 1) { // 11pm - 1am
	greeting = greetingsNight[randomGreeting(greetingsNight)];
} else if (hour < 12) { // Before noon (5am - 12pm)
	greeting = greetingsMorning[randomGreeting(greetingsMorning)];
} else if (hour >= 12 && hour <= 17) { // 12pm - 5pm
	greeting = greetingsAfternoon[randomGreeting(greetingsAfternoon)];
} else if (hour > 17 && hour < 23) { // 5pm - 11pm
	greeting = greetingsEvening[randomGreeting(greetingsEvening)];
} 

// Overwrite greeting if calculated holiday
if (holidaysByKey[holidayKey]) {
	greeting = holidaysByKey[holidayKey];
}

// Overwrite all greetings if specific holiday
if (holidaysByDate[holidayKeyDate]) {
	greeting = holidaysByDate[holidayKeyDate];
}

// Try/catch for color input parameter
try {
	inputArr[2].toString();
} catch(e) {
	throw new Error("Please long press the widget and add a parameter.");
}

let themeColor = new Color(inputArr[2].toString());

/* --------------- */
/* Assemble Widget */
/* --------------- */
 
//Top spacing
widgetHello.addSpacer(parseInt(spacing));
 
                                                                       
let shadow = new Color('#A9A9A9')
                                                                       
// Greeting label
let hello = widgetHello.addText(greeting);
hello.font = Font.boldSystemFont(29);
hello.textColor = themeColor;
hello.shadowColor = shadow;
hello.shadowOffset = new Point(1,1);
hello.shadowRadius = 2;
hello.textOpacity = (1);
                                                                       
hello.centerAlignText();
 
widgetHello.addSpacer(10);
 
let hStack = widgetHello.addStack();
hStack.layoutHorizontally();

// Centers weather line
hStack.addSpacer(45);
 
// Date label in stack
let datetext = hStack.addText(datefull + '\xa0\xa0\xa0\xa0');
datetext.font = Font.boldSystemFont(18);
datetext.textColor = themeColor;
datetext.shadowColor = shadow;
datetext.shadowOffset = new Point(1,1);
datetext.shadowRadius = 1;
datetext.textOpacity = (0.9);
datetext.centerAlignText();
 
//image
var tempImg = Image.fromFile(await fetchimagelocal(iconData + "_ico"));
 
//image in stack
let widgetimg = hStack.addImage(tempImg);
widgetimg.imageSize = new Size(20, 20);
widgetimg.shadowColor = shadow;
widgetimg.shadowOffset = new Point(1,1);
widgetimg.shadowRadius = 1;
widgetimg.imageOpacity = (0.9);
widgetimg.centerAlignImage();
 
//tempeture label in stack
let temptext = hStack.addText('\xa0\xa0'+ Math.round(curTemp).toString()+"\u2103");
temptext.font = Font.boldSystemFont(18);
temptext.textColor = themeColor;
temptext.shadowColor = shadow;
temptext.shadowOffset = new Point(1,1);
temptext.shadowRadius = 1;
temptext.textOpacity = (0.9);
temptext.centerAlignText();


// Countdown
const endtime = countdownTo;
function getTimeRemaining(endtime){
const total = Date.parse(endtime) - Date.parse(new Date());
const seconds = Math.floor( (total/1000) % 60 );
const minutes = Math.floor( (total/1000/60) % 60 );
const hours = Math.floor( (total/(1000*60*60)) % 24 );
const days = Math.floor( total/(1000*60*60*24) );

return {
total,
days,
hours,
minutes,
seconds
};
}

const total = Date.parse(endtime) - Date.parse(new Date());
let progressText = widgetHello.addText(String(getTimeRemaining(endtime).days + 1) + ' days until ' + countdownTitle)
progressText.font = Font.boldSystemFont(18);
progressText.textColor = themeColor;
progressText.shadowColor = shadow;
progressText.shadowOffset = new Point(1,1);
progressText.shadowRadius = 1;
progressText.textOpacity = (0.9);
progressText.centerAlignText();
 
//Battery
const batteryLine = widgetHello.addText(renderBattery());
batteryLine.textColor = themeColor;
batteryLine.font = Font.boldSystemFont(18);
batteryLine.shadowColor = shadow;
batteryLine.shadowOffset = new Point(1,1);
batteryLine.shadowRadius = 1;
batteryLine.shadowOpacity = (0.7);
batteryLine.textOpacity = (0.9);

function renderBattery() {
const batteryLevel = Device.batteryLevel();
const batteryAscii = "⚡️" + Math.round(batteryLevel * 100) + "%";
return batteryAscii;
}
                        
batteryLine.centerAlignText();

 
// Bottom Spacer
widgetHello.addSpacer();
widgetHello.setPadding(0, 0, 0, 0);
 
// Background image
widgetHello.backgroundImage = Image.fromFile(backgroundImageURL);
                        
// Set widget
Script.setWidget(widgetHello);