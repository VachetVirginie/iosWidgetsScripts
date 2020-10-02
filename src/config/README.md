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
}

````