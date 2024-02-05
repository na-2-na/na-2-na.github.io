---
title: "주식시황 자동으로 읽어오는 코드"
excerpt: "데일리로 포스트 가능한 컨텐츠-주식시황"
categories:
  - IT
tags:
  - Python
  - crawling
date: 2024-02-01
sidebar:
  nav: "sidebar-category"
---
<div oncopy="return false;">

네이버에 주식시황을 볼 수 있는 페이지가 있다.

    # 네이버 금융 리서치 페이지 URL
    url = "https://finance.naver.com/research/company_list.naver"


이 페이지를 통해 종목 별 내용을 가지고 보여주는 페이지를 만들고 싶었다.

저 페이지에서는 증권사별로 나래비(?) 되어있는데,  
종목별로 묶어서 보여주면 더 편하지 않을까 싶어서다.

코드 구현은 python 으로 했다.  
패키지 설치도 간단하고 환경설정이랄게 없다.

GPT에게 여러번 질문하여 잘 수행되는 코드를 받았고,  
마크다운의 형식대로 작성하는 부분까지 물어봐서 잘 조인해서 코드를 만들었다.

생각보다 잘돌아가고  
이코드를 Daily로 수행해서 게시하려고 한다.

문제점은 이 페이지를 google indexing 요청하면 바로 처리되지 않는다는거..

우선 이 시황 페이지는 당일 날짜는 검색이 안될테니  
나를 위한 페이지?가 아닐까 싶다.

주식시황만 긁어오는 부분을 코드로 남겨본다.


<div class="no-copy">

```python

def get_stock_research(title, date, author, categories=None, tags=None):

  # front matter
  front_matter = f'''---
title: "주식시황-{date[:10]}"
excerpt : "자동으로 읽어오는 주식시황"
categories:
    - STOCK
tags:
    - 주식
date: {date[:10]}
sidebar: 
    nav: "sidebar-category"
---
'''

  markdown_content = front_matter +'\n'

  # 네이버 금융 리서치 페이지 URL
  url = "https://finance.naver.com/research/company_list.naver"

  # HTTP GET 요청을 보냅니다.
  response = requests.get(url)

  # HTTP 요청이 성공했는지 확인합니다.
  if response.status_code == 200:
    # BeautifulSoup을 사용하여 HTML 문서를 파싱합니다.
    soup = BeautifulSoup(response.text, 'html.parser')

    # 오늘 날짜를 가져옵니다.
    today_date = datetime.now().strftime('%Y-%m-%d')
    #unix_time = int(time.time())

    # Markdown 파일로 저장할 내용을 담을 변수를 초기화합니다.
    markdown_content += f"# Stock Research - {today_date}\n\n"

    # 종목명과 증권사 정보를 추출합니다.
    research_list = soup.find_all('table', class_='type_1')

    # 종목별로 정보를 저장할 딕셔너리를 초기화합니다.
    stock_dict = {}

    for research_table in research_list:
      tr_list = research_table.find_all('tr')

      for tr in tr_list[1:]:
        td_list = tr.find_all('td')

        # 데이터가 충분한지 확인합니다.
        if len(td_list) >= 2:
          # 종목명과 증권사 정보를 추출합니다.
          stock_name = td_list[0].text.strip()
          security_firm = td_list[1].text.strip()

          # 종목명을 키로 증권사 정보를 딕셔너리에 추가합니다.
          if stock_name in stock_dict:
            stock_dict[stock_name].append(security_firm)
          else:
            stock_dict[stock_name] = [security_firm]
        else:
          print("데이터가 충분하지 않습니다.")

    # 종목별로 Markdown 형식으로 내용을 추가합니다.

    for stock_name, security_firms in stock_dict.items():
      markdown_content += f"## {stock_name}\n"
      markdown_content += "- " + "\n- ".join(security_firms) + "\n\n"

    create_markdown_file(markdown_content, today_date)
    # # Markdown 파일로 저장합니다.
    # with open('stock_research.md', 'w', encoding='utf-8') as f:
    #     f.write(markdown_content)
    #
    # print("Markdown 파일이 생성되었습니다.")
  else:
    print("HTTP 요청에 실패하였습니다.")


```

</div>