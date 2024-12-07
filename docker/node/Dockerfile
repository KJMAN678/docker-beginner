FROM node

ENV NODE_ENV=production

WORKDIR /scripts

COPY package* .

RUN <<EOF
npm install
groupadd app
useradd -g app -m app
mv /root/.config /home/app/ \
chown -R app:app /scripts /home/app/.config
EOF

COPY . .

RUN npm run build

USER app

EXPOSE 3000

CMD ["npm", "run", "start"]
