# ROYGBIV

![Prism Preview](https://i.imgur.com/heUckq4.jpg)

ROYGBIV creates Lightning prisms, which are split Lightning payments to different parties at percentages you define.

This repository contains a progressive web app - the frontend compontent to ROYGBIV. The backend Core Lightning plugin is available [here](https://github.com/daGoodenough/ROYGBIV-backend) and requires Core Lightning with the runtime flag 'experimental-dual-fund.' 

ROYGBIV prisms are a BOLT 12 offers, which are static QR codes that can be reused again and again. The output of a prism splits payments to other BOLT 12 offers, Lightning Addresses, or node pubkeys via keysend. 

## Install Instructions:

1. Clone (or fork) ROYGBIV:

```bash
git clone https://github.com/johngribbin/ROYGBIV-frontend.git
cd ROYGBIV-frontend
```

2. Install the dependencies

```bash
npm i
```

3. Start the app in dev mode

```bash
npm run dev
```

4. Open up [http://localhost:5173](http://localhost:5173) in your browser

5. Follow the instructions at [https://github.com/daGoodenough/ROYGBIV-backend](https://github.com/daGoodenough/ROYGBIV-backend) to set up the backend. 
