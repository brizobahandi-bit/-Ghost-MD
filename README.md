import makeWASocket, {
  useMultiFileAuthState,
  DisconnectReason
} from '@whiskeysockets/baileys';

async function startGhostMD() {
  const { state, saveCreds } =
    await useMultiFileAuthState('./session');

  const sock = makeWASocket({
    auth: state,
    printQRInTerminal: true,
    markOnlineOnConnect: false
  });

  sock.ev.on('creds.update', saveCreds);

  sock.ev.on('connection.update', (update) => {
    const { connection, lastDisconnect } = update;

    if (connection === 'open') {
      console.log('👻 GHOST MD CONNECTED!');
    }

    if (connection === 'close') {
      const code = lastDisconnect?.error?.output?.statusCode;

      if (code !== DisconnectReason.loggedOut) {
        startGhostMD();
      }
    }
  });

  sock.ev.on('messages.upsert', async ({ messages }) => {
    const msg = messages[0];

    if (!msg.message || msg.key.fromMe) return;

    const jid = msg.key.remoteJid;
    const body =
      msg.message.conversation ||
      msg.message.extendedTextMessage?.text ||
      '';

    if (body.toLowerCase() === '!ping') {
      await sock.sendMessage(jid, {
        text: '👻 Ghost MD est en ligne !'
      });
    }

    if (body.toLowerCase() === '!menu') {
      await sock.sendMessage(jid, {
        text: `👻 GHOST MD

📌 COMMANDES DISPONIBLES

!ping
!menu
!owner

🔥 Ghost MD - Multi Device`
      });
    }

    if (body.toLowerCase() === '!owner') {
      await sock.sendMessage(jid, {
        text: '👑 Propriétaire : Ghost MD'
      });
    }
  });
}

startGhostMD();
