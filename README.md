spider
======

sakshi newspaper top story spider

integrated_data = hxs.xpath('//p[@class="caption"]')
		for news_data in integrated_data:
			
			title_list = news_data.xpath("a/text()").extract()
			link = news_data.xpath("a/@href").extract()
			snippet=news_data.xpath("span/text()").extract()
			
			news_title=title_list[0].encode('utf-8')
			print news_title
			print link[0]
			print snippet[0].encode('utf-8')
