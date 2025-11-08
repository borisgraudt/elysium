# Elysium Web Specification

## Overview

Elysium Web enables decentralized websites hosted on the MeshNet network, accessible without internet infrastructure.

## Site Protocol

### GET_SITE
Request a site from the network:
```
GET_SITE <site_id>
```

### SEND_SITE
Broadcast a site to peers:
```
SEND_SITE <site_id> <html_content>
```

## Site Structure

Sites are stored in `sites/<site_id>/`:
```
site_id/
├── index.html
├── style.css
├── script.js
└── assets/
```

## Caching

Nodes cache sites locally for offline access and faster delivery.

## Site Discovery

Sites are discovered through:
1. Direct peer requests
2. Broadcast announcements
3. DHT (future implementation)

## Web Interface

The web dashboard provides:
- Network topology visualization
- Real-time chat
- Site browser
- Peer statistics
- AI routing visualization

