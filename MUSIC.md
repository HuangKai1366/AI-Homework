# AI-Homework
# import modules

import requests
from bs4 import BeautifulSoup
import webbrowser

def search_billboard(chart_type='hot-100'):
    if chart_type == 'global-200':
        url = 'https://www.billboard.com/charts/billboard-global-200/'
    else:
        url = 'https://www.billboard.com/charts/hot-100/'
    headers = {'User-Agent': 'Mozilla/5.0'}
    response = requests.get(url, headers=headers)
    if response.status_code != 200:
        return []
    soup = BeautifulSoup(response.text, 'html.parser')
    songs = []
    for item in soup.select('li.o-chart-results-list__item'):
        title_tag = item.select_one('h3')
        artist_tag = item.select_one('span.c-label')
        if title_tag and artist_tag:
            title = title_tag.get_text(strip=True)
            artist = artist_tag.get_text(strip=True)
            yt_query = f"https://www.youtube.com/results?search_query={title.replace(' ', '+')}+{artist.replace(' ', '+')}"
            songs.append({'title': title, 'artist': artist, 'url': yt_query})
        if len(songs) >= 10:
            break
    return songs


def main():
    print('請選擇排行榜來源：')
    print('1. Billboard Hot 100（美國單曲榜，僅統計美國地區）')
    print('2. Billboard Global 200（全球單曲榜，統計全球數據）')
    print('說明：Hot 100 代表美國本地最熱門單曲，Global 200 則是全球熱門單曲。')
    choice = input('請輸入 1 或 2：')
    if choice == '2':
        print('正在搜尋Billboard Global 200 TOP10...')
        songs = search_billboard('global-200')
        chart_name = 'Billboard Global 200'
    else:
        print('正在搜尋Billboard Hot 100 TOP10...')
        songs = search_billboard('hot-100')
        chart_name = 'Billboard Hot 100'
    if not songs:
        print('找不到相關歌曲。')
        return
    print(f'\n本週{chart_name} TOP10歌曲：')
    for idx, song in enumerate(songs, 1):
        print(f'{idx}. {song["title"]} - {song["artist"]} 連結: {song["url"]}')
    print('\n請輸入要播放的歌曲編號（可用逗號分隔多首，如 1,3,5），或直接按 Enter 跳過：')
    play_input = input('你的選擇：')
    if play_input.strip():
        try:
            indices = [int(i.strip()) for i in play_input.split(',') if i.strip().isdigit()]
            for idx in indices:
                if 1 <= idx <= len(songs):
                    webbrowser.open(songs[idx-1]['url'])
        except Exception as e:
            print('輸入格式錯誤，請輸入正確的編號。')
    else:
        print('已結束，不播放。')

if __name__ == '__main__':
    main()
