from flask import Flask, request, jsonify
import requests
from bs4 import BeautifulSoup

app = Flask(__name__)

@app.route('/search')
def search_images():
    query = request.args.get('query')
    if not query:
        return jsonify({'error': 'No query provided'})

    url = f"https://lexica.art/?q={query}"
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
        "Referer": "https://lexica.art"
    }

    response = requests.get(url, headers=headers)
    soup = BeautifulSoup(response.content, 'html.parser')

    twitter_image_meta = soup.find('meta', attrs={'name': 'twitter:image'})
    if not twitter_image_meta:
        return jsonify({'error': 'No images found for the query'})

    content_value = twitter_image_meta.get('content', '')
    image_urls = content_value.split('&images=')
    
    image_urls = [image_url for image_url in image_urls if image_url.startswith("https://image.lexica.art/")]

    return jsonify({'images': image_urls})

if __name__ == '__main__':
    app.run(debug=True)
