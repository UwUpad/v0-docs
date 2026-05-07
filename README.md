# 0pad (uwupad.me) API Documentation

Welcome to the **0pad API**. This API allows you to programmatically interact with the uwupad platform to search for sounds, retrieve categories, manage playlists, and download audio files. 

## Base Information

* **Base URL:** `https://v0.uwupad.me/api/v1`
* **Content-Type:** `application/json; charset=utf-8`

## Authentication

All programmatic access via third-party apps, scripts, or bots **requires an API Token**. 
You can generate one in your account settings under the **Developer API** tab on the website.

Pass your token in the `X-API-Key` header with every request:
```http
GET /api/v1/sounds HTTP/1.1
Host: v0.uwupad.me
X-API-Key: sk_0pad_your_token_here
```

## Rate Limits

Rate limits are enforced based on your account plan. Limits are applied per minute.
* **Free Plan:** 60 requests / minute
* **Business Plan:** 1000 requests / minute

You can monitor your limits via HTTP response headers:
* `X-RateLimit-Limit`: Maximum requests allowed per minute.
* `X-RateLimit-Remaining`: Requests remaining in the current window.

If you exceed the limit, you will receive a `429 Too Many Requests` response.

---

## 1. Sounds Endpoints

### 1.1 Search and List Sounds
`GET /sounds`

Retrieves a paginated list of sounds based on search queries, filters, and sorting.

**Query Parameters:**

| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `page` | int | `1` | Page number. |
| `limit` | int | `30` | Items per page (max 100). |
| `sort` | string | `fyp` | Sorting field: `fyp`, `date`, `title`, `duration`, `downloads`, `views`, `likes`, `comments`. |
| `order` | string | `desc` | Sorting direction: `desc` or `asc`. |
| `q` | string | | Global search query (smart search across title and tags). |
| `category` | string | `all` | Filter by category slug (e.g., `memes`, `sfx`, `gaming`). |
| `smart` | int | `1` | Smart search toggle. `1` = tokenized relevance search, `0` = strict search. |
| `title_mode` | string | `contains` | Search mode for title: `contains`, `exact`, `starts`, `ends`. |
| `tag_mode` | string | `contains` | Search mode for tags. |
| `desc_mode`| string | `contains` | Search mode for description. |
| `tag` | string | | Search specifically by tag. |
| `desc` | string | | Search specifically inside descriptions. |
| `pos` | string | | CSV of words that MUST be in the title (e.g., `cat,meow`). |
| `neg` | string | | CSV of words that MUST NOT be in the title (e.g., `loud,bass`). |
| `explicit` | string | | `1` to show only explicit, `0` to show only clean. |
| `loud` | string | | `1` to show only loud sounds, `0` to exclude loud sounds. |
| `langs` | string | | CSV of language codes (e.g., `en,ru,sfx`). |
| `dur_from` | float | | Minimum duration in seconds. |
| `dur_to` | float | | Maximum duration in seconds. |
| `vw_from`, `vw_to` | int | | Views range filter. |
| `dl_from`, `dl_to` | int | | Downloads range filter. |
| `lk_from`, `lk_to` | int | | Likes range filter. |
| `cm_from`, `cm_to` | int | | Comments range filter. |
| `user_id` | int | | Filter sounds uploaded by a specific user. |
| `playlist_id`| int | | Get sounds belonging to a specific playlist. |

**Response:**
Returns a `SoundsListResponse` object. Also includes an `X-Total-Count` header.

```json
{
  "total": 1500,
  "page": 1,
  "limit": 30,
  "data":[
    {
      "id": 12345,
      "title": "Funny Cat Meow",
      "ext": "mp3",
      "dur": 2.5,
      "loud": false,
      "expl": false,
      "dl": 120,
      "lk": 45,
      "cm": 2,
      "vw": 1500,
      "ts": 1715000000,
      "lang": "en",
      "category": "animals",
      "owner": {
        "id": 1,
        "name": "daisov",
        "av": "https://cdn.uwupad.me/avatars/1/48.webp",
        "role": "admin"
      },
      "tags":[
        {"id": 10, "name": "cat"},
        {"id": 12, "name": "meow"}
      ]
    }
  ]
}
```

### 1.2 Get Sound Details
`GET /sounds/{id}`

Retrieves detailed information about a specific sound.

**Response:**
Returns a `SoundDetailResponse` object.

```json
{
  "id": 12345,
  "title": "Funny Cat Meow",
  "ext": "mp3",
  "dur": 2.5,
  "dl": 120,
  "lk": 45,
  "cm": 2,
  "vw": 1500,
  "ts": 1715000000,
  "desc": "Just a funny cat meow sound I recorded.",
  "is_orig": true,
  "owner": {
    "id": 1,
    "name": "daisov"
  },
  "tags":[
    {"id": 10, "name": "cat"}
  ]
}
```

---

## 2. Playlists Endpoints

### 2.1 Search and List Playlists
`GET /playlists`

**Query Parameters:**
| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `page`, `limit` | int | `1`, `30`| Pagination. |
| `q` | string | | Search playlist by name. |
| `sort` | string | `latest` | `latest`, `views`, `soundcount`, `name`. |
| `order` | string | `desc` | `desc` or `asc`. |
| `user_id` | int | | Playlists created by specific user. |
| `vw_from`, `vw_to` | int | | View count filter. |
| `sc_from`, `sc_to` | int | | Sound count filter. |

**Response:**
Returns a `PlaylistsListResponse` object.

### 2.2 Get Playlist Details
`GET /playlists/{id}`

**Response:**
Returns a `PlaylistResponse` object.

---

## 3. Users Endpoints

### 3.1 Get User Profile
`GET /users/{id}`

Retrieves public profile statistics for a user.

**Response:**
```json
{
  "id": 1,
  "name": "daisov",
  "av": "https://cdn.uwupad.me/avatars/1/48.webp",
  "role": "admin",
  "influence": 5400,
  "total_views": 150000,
  "total_downloads": 40000,
  "total_uploads": 120,
  "total_likes": 3000
}
```

### 3.2 Search Users
`GET /users/search?q={query}`

Search for users by username. Returns max 10 results.

---

## 4. Download / Play Audio
`GET /download/{id}`
or
`GET /download/{id}/{filename}` *(Recommended for Soundpad integration)*

This endpoint streams the audio file directly from the CDN with proper `Content-Disposition` headers so the browser or application knows the real filename and extension. It also increments the download counter.

**Important for Custom Players / Soundpad Integrations:**
If you want a user to directly add a sound to **Soundpad** from your app without downloading it to their disk manually, use the native Soundpad URL protocol.

*Format:* `soundpad://sound/url/https://v0.uwupad.me/api/v1/download/{id}/{safetitle}.{ext}`

*Note: You must pass your `X-API-Key` in headers if you are fetching the audio binary directly via code, otherwise access will be forbidden (CORS/Bot protection).*

---

## 5. Data Models Dictionary

**OwnerMini**
| Field | Type | Description |
| :--- | :--- | :--- |
| `id` | int | User ID. |
| `name` | string | Username. |
| `av` | string | CDN URL to the 48px avatar (omitted if null). |
| `role` | string | E.g. `admin`, `moderator` (omitted if standard `user`). |

**TagMini**
| Field | Type | Description |
| :--- | :--- | :--- |
| `id` | int | Tag ID. |
| `name` | string | Tag name (e.g. `memes`). |

**SoundResponse** (Object inside `SoundsListResponse.data`)
| Field | Type | Description |
| :--- | :--- | :--- |
| `id` | int | Unique Sound ID. |
| `title` | string | Sound title. |
| `ext` | string | File extension (`mp3`, `ogg`, etc). |
| `dur` | float | Duration in seconds. |
| `loud` | bool | True if flagged as loud. |
| `expl` | bool | True if flagged as explicit/NSFW. |
| `dl` | int | Total downloads. |
| `lk` | int | Total likes. |
| `cm` | int | Total comments. |
| `vw` | int | Total views/plays. |
| `ts` | int64 | Unix timestamp of upload. |
| `lang` | string | Language code (e.g., `en`, `ru`, `sfx`). |
| `category` | string | Category slug. |
| `owner` | OwnerMini | Uploader info. |
| `tags` | array | Array of `TagMini` objects. |

---

## 6. Code Examples

### Node.js (Axios)
```javascript
const axios = require('axios');

const API_KEY = 'sk_0pad_your_token_here';
const BASE_URL = 'https://v0.uwupad.me/api/v1';

async function fetchTrendingMemes() {
  try {
    const response = await axios.get(`${BASE_URL}/sounds`, {
      headers: { 'X-API-Key': API_KEY },
      params: {
        category: 'memes',
        sort: 'fyp',
        limit: 10
      }
    });
    
    const sounds = response.data.data;
    sounds.forEach(s => {
      console.log(`[${s.dur}s] ${s.title} by ${s.owner.name}`);
      // For Soundpad integration URL:
      const safeTitle = encodeURIComponent(s.title.replace(/\s+/g, '_'));
      const spUrl = `soundpad://sound/url/${BASE_URL}/download/${s.id}/${safeTitle}.${s.ext}`;
      console.log(`-> Add to Soundpad: ${spUrl}`);
    });

  } catch (error) {
    console.error('API Error:', error.response ? error.response.data : error.message);
  }
}

fetchTrendingMemes();
```

### Python (Requests)
```python
import requests
import urllib.parse

API_KEY = "sk_0pad_your_token_here"
BASE_URL = "https://v0.uwupad.me/api/v1"

headers = {
    "X-API-Key": API_KEY
}

# 1. Search for a specific sound
params = {
    "q": "bruh effect",
    "smart": 1,
    "limit": 5
}

response = requests.get(f"{BASE_URL}/sounds", headers=headers, params=params)

if response.status_code == 200:
    data = response.json()
    for sound in data.get("data", []):
        print(f"Found: {sound['title']} (ID: {sound['id']})")
        
        # 2. Generate a Download / Soundpad URL
        safe_title = urllib.parse.quote(sound['title'].replace(" ", "_"))
        dl_url = f"{BASE_URL}/download/{sound['id']}/{safe_title}.{sound['ext']}"
        print(f"Direct download link: {dl_url}\n")
else:
    print("Error:", response.json())
```

### Go
```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	req, _ := http.NewRequest("GET", "https://v0.uwupad.me/api/v1/sounds?q=cat&limit=5", nil)
	req.Header.Add("X-API-Key", "sk_0pad_your_token_here")

	res, err := http.DefaultClient.Do(req)
	if err != nil {
		fmt.Println("Request error:", err)
		return
	}
	defer res.Body.Close()

	body, _ := ioutil.ReadAll(res.Body)
	fmt.Println(string(body))
}
```
