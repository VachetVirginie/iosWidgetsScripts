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

## random comic

````
// Variables used by Scriptable.
// These must be at the very top of the file. Do not edit.
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