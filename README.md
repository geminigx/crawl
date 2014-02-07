crawl
=====

import scala.xml.XML
import scala.io.Source
import java.io.{IOException, FileNotFoundException}


object Reader {

  val wasPriceHtmlString = "<span class='pp-was-price'>"
  val startH1 = "<h1"
  val endH1 = "</h1>"
  val startSpan = "<span>"
  val endSpan = "</span>"

  def netParse(sUrl: String): String = {
    Source.fromURL(sUrl,"ISO-8859-1").mkString
  }

  def getUniqueName(wholeHtml: String, start: String, end: String): String = {
    val titleSubString = wholeHtml.substring(wholeHtml.indexOf(start))
    val xmlShortString = titleSubString.substring(0, titleSubString.indexOf(end) + end.size).replace("&","")
    XML.loadString(xmlShortString).text.trim
  }

  def cleanPrice(price: String): String ={
    if(price.contains("-")){
      price.substring(0, price.indexOf("-")-1)
    }else
      price
  }

  def processOneURL(url: String) {
    val wholeHtml = netParse(url)
    val wasPriceIndex = wholeHtml.indexOf(wasPriceHtmlString)
    if (!wholeHtml.contains("Sorry, this item is currently unavailable") && wasPriceIndex != -1) {
      val title = getUniqueName(wholeHtml, startH1, endH1)
      val wasPrice = cleanPrice(getUniqueName(wholeHtml, wasPriceHtmlString, endSpan))
      val nowPrice = cleanPrice(getUniqueName(wholeHtml.substring(wasPriceIndex), startSpan, endSpan))
      val pctOff = 1.0 - nowPrice.replace("NOW $", "").toDouble / wasPrice.replace("$", "").toDouble

      if (pctOff > 0.8)
        println(title + " - " + pctOff * 100 + "% - " + url)
    }
  }

  def main(args: Array[String]) {
    println("Hello, world!")

    processOneURL("http://www.landsend.com/products/mens-wool-down-parka/id_260920")

    val fixedURL = "http://www.landsend.com/products/mens-wool-down-parka/id_"
    //started with 260000
    val startingIndex = 252000

    (0 until 1000).foreach(
      x => {
        val newUrl = fixedURL + (startingIndex + x)
        try{
          processOneURL(newUrl)
          Thread.sleep(1000)
        }catch {
          case e: FileNotFoundException => ""
          case e: IOException => ""
          case e: Exception =>
            println(newUrl)
            e.printStackTrace()
        }
      }
    )
  }
}
