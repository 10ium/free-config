export default {
  async fetch(request) {
    const url = new URL(request.url);

    // ۱. سرویس فاویکون مستقل و بدون پردازش
    if (url.pathname === "/favicon.ico") {
      const faviconBase64 = "iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAAAhklEQVQ4T2NkYGD4z0ABYBw1gGE0DBhGwwBhNAwYRsOAYTQMGEYDAGzQn0H6qCgMAAAAAElFTkSuQmCC";
      return new Response(Uint8Array.from(atob(faviconBase64), c => c.charCodeAt(0)), {
        headers: { "Content-Type": "image/png", "Cache-Control": "public, max-age=86400" }
      });
    }

    // ۲. رابط کاربری (UI) حرفه‌ای
    if (url.pathname === "/" || url.pathname === "") {
      return new Response(getHTML(url.origin), {
        headers: { "Content-Type": "text/html; charset=utf-8" }
      });
    }

    // ۳. هسته پردازشی و تولید خروجی (با موتور توربو)
    if (url.pathname === "/mihomo") {
      try {
        const customSubsRaw = url.searchParams.get("subs");
        let SUBS = [];
        
        if (customSubsRaw) {
          SUBS = [...new Set(customSubsRaw.split(",").map(s => s.trim()).filter(Boolean))];
        } else {
          SUBS = [...new Set(`
          https://raw.githubusercontent.com/10ium/VpnClashFaCollector/main/sub/tested/ping_passed.txt
          https://raw.githubusercontent.com/10ium/multi-proxy-config-fetcher/refs/heads/main/configs/proxy_configs.txt
          https://raw.githubusercontent.com/10ium/HiN-VPN/main/subscription/normal/mix
          https://raw.githubusercontent.com/10ium/V2Hub3/main/merged
          https://raw.githubusercontent.com/10ium/V2RayAggregator/refs/heads/master/Eternity.txt
          https://raw.githubusercontent.com/10ium/telegram-configs-collector/main/protocols/hysteria
          https://raw.githubusercontent.com/10ium/telegram-configs-collector/main/protocols/reality
          https://raw.githubusercontent.com/10ium/telegram-configs-collector/main/countries/ir/mixed
          https://raw.githubusercontent.com/10ium/telegram-configs-collector/main/countries/ae/mixed
          https://raw.githubusercontent.com/10ium/telegram-configs-collector/main/countries/tr/mixed
          https://raw.githubusercontent.com/10ium/telegram-configs-collector/main/countries/om/mixed
          https://raw.githubusercontent.com/10ium/telegram-configs-collector/main/countries/jo/mixed
          https://raw.githubusercontent.com/10ium/telegram-configs-collector/main/countries/iq/mixed
          https://raw.githubusercontent.com/10ium/telegram-configs-collector/main/countries/hu/mixed
          https://raw.githubusercontent.com/10ium/telegram-configs-collector/main/countries/gr/mixed
          https://raw.githubusercontent.com/10ium/telegram-configs-collector/main/countries/bh/mixed
          https://raw.githubusercontent.com/10ium/telegram-configs-collector/main/countries/am/mixed
          https://raw.githubusercontent.com/roosterkid/openproxylist/refs/heads/main/V2RAY_RAW.txt
          https://raw.githubusercontent.com/Rayan-Config/C-Sub/refs/heads/main/configs/proxy.txt
          https://raw.githubusercontent.com/Freedom-Guard-Builder/Freedom-Finder/refs/heads/main/out/mixed_configs.txt
          https://raw.githubusercontent.com/MahsaNetConfigTopic/config/refs/heads/main/xray_final.txt
          https://raw.githubusercontent.com/mahsanet/MahsaFreeConfig/refs/heads/main/mtn/sub_1.txt
          https://raw.githubusercontent.com/mahsanet/MahsaFreeConfig/refs/heads/main/mtn/sub_2.txt
          https://raw.githubusercontent.com/mahsanet/MahsaFreeConfig/refs/heads/main/mci/sub_2.txt
          https://raw.githubusercontent.com/mahsanet/MahsaFreeConfig/refs/heads/main/mci/sub_1.txt
          https://raw.githubusercontent.com/maimengmeng/mysub/refs/heads/main/valid_content.txt
          https://raw.githubusercontent.com/itsyebekhe/PSG/main/subscriptions/xray/base64/reality
          https://raw.githubusercontent.com/itsyebekhe/PSG/main/subscriptions/xray/base64/xhttp
          https://raw.githubusercontent.com/itsyebekhe/PSG/refs/heads/main/subscriptions/xray/normal/hy2
          https://raw.githubusercontent.com/itsyebekhe/PSG/refs/heads/main/subscriptions/xray/normal/openai
          https://raw.githubusercontent.com/itsyebekhe/PSG/refs/heads/main/subscriptions/locations/normal/IR
          https://raw.githubusercontent.com/hamedp-71/N_sub_cheker/refs/heads/patch-1/final.txt
          https://raw.githubusercontent.com/MrBihal/Channel-Hddify/refs/heads/main/MeLi-Shekan
          https://raw.githubusercontent.com/MrBihal/Channel-Hddify/refs/heads/main/Meli
          https://raw.githubusercontent.com/igareck/vpn-configs-for-russia/refs/heads/main/BLACK_SS%2BAll_RUS.txt
          https://raw.githubusercontent.com/igareck/vpn-configs-for-russia/refs/heads/main/BLACK_VLESS_RUS.txt
          https://raw.githubusercontent.com/igareck/vpn-configs-for-russia/refs/heads/main/BLACK_VLESS_RUS_mobile.txt
          https://raw.githubusercontent.com/peweza/PUBLICSUB/refs/heads/main/PewezaVPNPubSUB
          https://raw.githubusercontent.com/frank-vpl/servers/refs/heads/main/irbox
          https://v2.alicivil.workers.dev/?list=mix&count=500&shuffle=false&unique=false
          `.split("\n").map(s => s.trim()).filter(Boolean))];
        }

        const fetchErrors = [];
        
        // بهینه‌سازی عظیم حافظه: استفاده از Map سراسری
        const uniqueProxies = new Map();
        
        // از پیش تعریف کردن Regex
        const CONTROL_CHAR_REGEX = /[\x00-\x1F\x7F-\x9F\u200B-\u200D\uFEFF\uFFFD]/;
        const REPLACE_REGEX = /[\x00-\x1F\x7F-\x9F\u200B-\u200D\uFEFF\uFFFD]/g;

        const fetchPromises = SUBS.map(async (sub) => {
          const controller = new AbortController();
          const timeout = setTimeout(() => controller.abort(), 8000); 
          try {
            const res = await fetch(sub, { signal: controller.signal, cf: { cacheTtl: 0 } });
            if (!res.ok) throw new Error(`HTTP Error: ${res.status}`);
            
            const raw = await res.text();
            const decoded = decodeSub(raw);
            const lines = decoded.split("\n");
            
            for (let i = 0; i < lines.length; i++) {
              let line = lines[i].trim();
              if (!line || line.startsWith("#")) continue;
              
              if (CONTROL_CHAR_REGEX.test(line)) {
                line = line.replace(REPLACE_REGEX, "");
              }
              
              let p = parseProxy(line);
              if (p) {
                p.name = p.name || "Unnamed";
                if (valid(p)) {
                  const fp = p.type + "|" + p.server + "|" + p.port + "|" + (p.uuid || p.password || p.auth_str || p["private-key"] || p.psk || p.user || "");
                  if (!uniqueProxies.has(fp)) {
                    uniqueProxies.set(fp, p);
                  }
                }
              }
            }
          } catch (error) {
            fetchErrors.push({ url: sub, reason: error.name === 'AbortError' ? 'Timeout (8s)' : error.message });
          } finally {
            clearTimeout(timeout);
          }
        });

        await Promise.allSettled(fetchPromises);

        let proxies = Array.from(uniqueProxies.values());

        const getDisplayType = (type) => {
          if (type === "hysteria2") return "hy2";
          if (type === "hysteria") return "hy";
          if (type === "wireguard") return "wg";
          if (type === "socks5") return "socks";
          return type.toLowerCase();
        };

        const protocolsList = ["hy2", "hy", "vless", "anytls", "trojan", "ss", "vmess", "wg", "tuic", "ssr", "snell", "http", "socks", "ssh"];
        
        const limits = { total: parseInt(url.searchParams.get("limit_total")) || Infinity };
        const priorities = {};
        
        protocolsList.forEach(t => {
          const limitVal = url.searchParams.get(`limit_${t}`);
          limits[t] = (limitVal !== null && limitVal !== "") ? parseInt(limitVal) : Infinity;
          
          const prioVal = url.searchParams.get(`prio_${t}`);
          priorities[t] = (prioVal !== null && prioVal !== "") ? parseInt(prioVal) : 99;
        });

        proxies.sort((a, b) => {
          const orderA = priorities[getDisplayType(a.type)] || 99;
          const orderB = priorities[getDisplayType(b.type)] || 99;
          return orderA - orderB;
        });

        const typeCounters = {};
        let filteredProxies = [];
        for (let i = 0; i < proxies.length; i++) {
          const p = proxies[i];
          const displayType = getDisplayType(p.type);
          
          if (limits[displayType] === 0) continue; 
          
          typeCounters[displayType] = (typeCounters[displayType] || 0) + 1;
          if (typeCounters[displayType] <= limits[displayType]) {
            filteredProxies.push(p);
          }
          
          if (filteredProxies.length >= limits.total) break; 
        }

        const namingCounters = {};
        for (let i = 0; i < filteredProxies.length; i++) {
          const p = filteredProxies[i];
          const displayType = getDisplayType(p.type);
          namingCounters[displayType] = (namingCounters[displayType] || 0) + 1;
          p.name = `${displayType} ${namingCounters[displayType]}`;
        }

        // اعمال ویژگی جدید: Amnezia WireGuard Option به پروکسی‌های WG در صورت فعال بودن
        if (url.searchParams.get("awg") === "1") {
          const awg_jc = parseInt(url.searchParams.get("awg_jc")) || 4;
          const awg_jmin = parseInt(url.searchParams.get("awg_jmin")) || 40;
          const awg_jmax = parseInt(url.searchParams.get("awg_jmax")) || 100;

          for (let i = 0; i < filteredProxies.length; i++) {
            if (filteredProxies[i].type === "wireguard") {
              filteredProxies[i]["amnezia-wg-option"] = {
                jc: awg_jc,
                jmin: awg_jmin,
                jmax: awg_jmax,
                s1: 0,
                s2: 0,
                h1: 1,
                h2: 2,
                h3: 3,
                h4: 4
              };
            }
          }
        }

        if (url.searchParams.get("format") === "json") {
          const stats = { total: filteredProxies.length, types: {}, errors: fetchErrors };
          filteredProxies.forEach(p => {
            const t = getDisplayType(p.type);
            stats.types[t] = (stats.types[t] || 0) + 1;
          });
          return new Response(JSON.stringify(stats), { headers: { "Content-Type": "application/json" } });
        }

        // بررسی نوع خروجی
        const outputFormat = url.searchParams.get("output") || "provider";

        if (outputFormat === "config") {
          return new Response(buildFullConfig(filteredProxies), {
            headers: { 
              "Content-Type": "text/yaml; charset=utf-8",
              "Content-Disposition": 'attachment; filename="config.yaml"'
            }
          });
        }

        return new Response(buildProvider(filteredProxies), {
          headers: { "Content-Type": "text/yaml; charset=utf-8" }
        });
      } catch (e) {
        return new Response("Error Processing Subscription: " + e.toString(), { status: 500 });
      }
    }
    return new Response("Endpoint Not Found", { status: 404 });
  }
};

/* ==========================================
   بخش توابع کمکی (مطابق با استانداردهای Mihomo)
   ========================================== */

function normalizeBase64(v) {
  v = v.trim();
  if (!v) return null;
  if (v.includes('-') || v.includes('_') || v.includes(' ')) {
    v = v.replace(/-/g, "+").replace(/_/g, "/").replace(/\s+/g, "");
  }
  const pad = v.length % 4;
  if (pad === 2) v += "==";
  else if (pad === 3) v += "=";
  else if (pad === 1) return null;
  
  try {
    const binString = atob(v);
    const bytes = new Uint8Array(binString.length);
    for (let i = 0; i < binString.length; i++) {
      bytes[i] = binString.charCodeAt(i);
    }
    return new TextDecoder().decode(bytes);
  } catch { return null; }
}

function decodeSub(text) {
  if (text.includes("://")) return text;
  const fixed = normalizeBase64(text);
  if (!fixed) return text;
  return fixed;
}

function safeDecode(str) {
  try { return decodeURIComponent(str); } catch { return str; }
}

function parseProxy(line) {
  try {
    const prefix = line.substring(0, 15).toLowerCase();
    if (prefix.startsWith("vless://")) return parseVless(line);
    if (prefix.startsWith("vmess://")) return parseVmess(line);
    if (prefix.startsWith("trojan://")) return parseTrojan(line);
    if (prefix.startsWith("anytls://")) return parseAnyTls(line);
    if (prefix.startsWith("ss://")) return parseSS(line);
    if (prefix.startsWith("ssr://")) return parseSSR(line);
    if (prefix.startsWith("hy2://") || prefix.startsWith("hysteria2://")) return parseHysteria2(line);
    if (prefix.startsWith("hysteria://")) return parseHysteria(line);
    if (prefix.startsWith("wg://") || prefix.startsWith("wireguard://")) return parseWireguard(line);
    if (prefix.startsWith("tuic://")) return parseTuic(line);
    if (prefix.startsWith("http://") || prefix.startsWith("https://")) return parseHttp(line);
    if (prefix.startsWith("socks://") || prefix.startsWith("socks5://")) return parseSocks(line);
    if (prefix.startsWith("snell://")) return parseSnell(line);
    if (prefix.startsWith("ssh://")) return parseSSH(line);
  } catch {}
  return null;
}

function parseVless(link) {
  const url = new URL(link.replace(/^vless:\/\//i, "http://"));
  const security = url.searchParams.get("security") || "";
  const proxy = { 
    name: safeDecode(url.hash.substring(1) || url.hostname), 
    type: "vless", 
    server: url.hostname, 
    port: parseInt(url.port), 
    uuid: url.username || "", 
    udp: true, // Mihomo standard
    tls: ["tls","reality"].includes(security), 
    network: url.searchParams.get("type") || "tcp" 
  };
  const sni = url.searchParams.get("sni"); if (sni) proxy.servername = sni;
  const fp = url.searchParams.get("fp"); if (fp) proxy["client-fingerprint"] = fp;
  const alpn = url.searchParams.get("alpn"); if (alpn) proxy.alpn = alpn.split(",");
  const flow = url.searchParams.get("flow"); if (flow) proxy.flow = flow;

  if (security === "reality") {
    proxy["reality-opts"] = { "public-key": url.searchParams.get("pbk") || "" };
    const sid = url.searchParams.get("sid");
    if (sid) proxy["reality-opts"]["short-id"] = sid;
  }

  if (proxy.network === "ws") {
    const path = url.searchParams.get("path"); const host = url.searchParams.get("host");
    if (path || host) { proxy["ws-opts"] = {}; if (path) proxy["ws-opts"].path = path; if (host) proxy["ws-opts"].headers = { Host: host }; }
  } else if (proxy.network === "grpc") {
    const serviceName = url.searchParams.get("serviceName");
    if (serviceName) proxy["grpc-opts"] = { "grpc-service-name": serviceName };
  }
  return proxy;
}

function parseVmess(link) {
  const fixed = normalizeBase64(link.replace(/^vmess:\/\//i, "")); if (!fixed) return null;
  const j = JSON.parse(fixed);
  return { 
    name: j.ps || j.add, 
    type: "vmess", 
    server: j.add, 
    port: parseInt(j.port), 
    uuid: j.id || "", 
    alterId: parseInt(j.aid) || 0, // Mihomo requirement
    cipher: j.scy || "auto",       // Mihomo requirement
    udp: true
  };
}

function parseTrojan(link) {
  const url = new URL(link.replace(/^trojan:\/\//i, "http://"));
  const proxy = { 
    name: safeDecode(url.hash.substring(1) || url.hostname), 
    type: "trojan", 
    server: url.hostname, 
    port: parseInt(url.port), 
    password: safeDecode(url.username) || "", 
    udp: true, // Mihomo requirement for modern configs
    tls: true, 
    network: url.searchParams.get("type") || "tcp" 
  };
  const sni = url.searchParams.get("sni"); if (sni) proxy.servername = sni;
  if (proxy.network === "ws") {
    const path = url.searchParams.get("path"), host = url.searchParams.get("host");
    if (path || host) { proxy["ws-opts"] = {}; if (path) proxy["ws-opts"].path = path; if (host) proxy["ws-opts"].headers = { Host: host }; }
  } else if (proxy.network === "grpc") {
    const serviceName = url.searchParams.get("serviceName"); if (serviceName) proxy["grpc-opts"] = { "grpc-service-name": serviceName };
  }
  return proxy;
}

function parseAnyTls(link) {
  const url = new URL(link.replace(/^anytls:\/\//i, "http://"));
  const proxy = { 
    name: safeDecode(url.hash.substring(1) || url.hostname), 
    type: "anytls", 
    server: url.hostname, 
    port: parseInt(url.port) 
  };
  const pass = safeDecode(url.username || url.password); if (pass) proxy.password = pass;
  const sni = url.searchParams.get("sni"); if (sni) proxy.servername = sni;
  const alpn = url.searchParams.get("alpn"); if (alpn) proxy.alpn = alpn.split(",");
  const fp = url.searchParams.get("fp") || url.searchParams.get("fingerprint"); if (fp) proxy["client-fingerprint"] = fp;
  return proxy;
}

function parseSS(link) {
  const raw = link.replace(/^ss:\/\//i,""); const [base, tag] = raw.split("#");
  let method, password, server, port;
  if (base.includes("@")) {
    const decodedAuth = normalizeBase64(base.split("@")[0]) || base.split("@")[0];
    [method, password] = decodedAuth.split(":"); [server, port] = base.split("@")[1].split(":");
  } else {
    const decodedFull = normalizeBase64(base); if (!decodedFull) return null;
    const [authPart, serverPart] = decodedFull.split("@"); if (!authPart || !serverPart) return null;
    [method, password] = authPart.split(":"); [server, port] = serverPart.split(":");
  }
  if (!server || !port || !method || !password) return null;
  return { 
    name: safeDecode(tag || server), 
    type: "ss", 
    server, 
    port: parseInt(port), 
    cipher: method, 
    password: password || "",
    udp: true 
  };
}

function parseSSR(link) {
  const decoded = normalizeBase64(link.replace(/^ssr:\/\//i, "")); if (!decoded) return null;
  const parts = decoded.split("/");
  const [server, port, protocol, method, obfs, b64pass] = parts[0].split(":");
  if (!server || !port || !method) return null;
  const proxy = { 
    name: "SSR", 
    type: "ssr", 
    server: server, 
    port: parseInt(port), 
    cipher: method, 
    password: normalizeBase64(b64pass) || b64pass || "", 
    protocol: protocol, 
    obfs: obfs 
  };
  if (parts[1]) {
    const params = new URLSearchParams(parts[1].replace(/^\?/, ""));
    const remarks = params.get("remarks"); if (remarks) proxy.name = safeDecode(normalizeBase64(remarks) || remarks);
    const obfsparam = params.get("obfsparam"); if (obfsparam) proxy["obfs-param"] = safeDecode(normalizeBase64(obfsparam) || obfsparam);
    const protoparam = params.get("protoparam"); if (protoparam) proxy["protocol-param"] = safeDecode(normalizeBase64(protoparam) || protoparam);
  }
  return proxy;
}

function parseHysteria2(link) {
  const url = new URL(link.replace(/^(hy2|hysteria2):\/\//i, "http://"));
  const proxy = { 
    name: safeDecode(url.hash.substring(1) || url.hostname), 
    type: "hysteria2", 
    server: url.hostname, 
    port: parseInt(url.port), 
    password: safeDecode(url.username) || "" 
  };
  const sni = url.searchParams.get("sni") || url.searchParams.get("peer"); if (sni) proxy.sni = sni;
  const insecure = url.searchParams.get("insecure"); if (insecure === "1" || insecure === "true") proxy["skip-cert-verify"] = true;
  const obfs = url.searchParams.get("obfs");
  if (obfs && obfs !== "none") { proxy.obfs = obfs; const obfsPass = url.searchParams.get("obfs-password"); if (obfsPass) proxy["obfs-password"] = obfsPass; }
  return proxy;
}

function parseHysteria(link) {
  const url = new URL(link.replace(/^hysteria:\/\//i, "http://"));
  const proxy = { 
    name: safeDecode(url.hash.substring(1) || url.hostname), 
    type: "hysteria", 
    server: url.hostname, 
    port: parseInt(url.port), 
    protocol: url.searchParams.get("protocol") || "udp", // Mihomo standard
    up: url.searchParams.get("up") || "50 Mbps", 
    down: url.searchParams.get("down") || "100 Mbps" 
  };
  const auth = url.searchParams.get("auth") || url.searchParams.get("obfsParam"); if (auth) proxy.auth_str = auth;
  const sni = url.searchParams.get("peer") || url.searchParams.get("sni"); if (sni) proxy.sni = sni;
  const insecure = url.searchParams.get("insecure"); if (insecure === "1" || insecure === "true") proxy["skip-cert-verify"] = true;
  const alpn = url.searchParams.get("alpn"); if (alpn) proxy.alpn = alpn.split(",");
  return proxy;
}

function parseWireguard(link) {
  const url = new URL(link.replace(/^(wg|wireguard):\/\//i, "http://"));
  const rawIp = url.searchParams.get("ip") || url.searchParams.get("address") || "10.0.0.1";
  
  // استخراج کلید عمومی و خصوصی با پوشش تمام حالت‌های ممکن کوچک و بزرگ (Case-Insensitive)
  let pubKey = url.searchParams.get("public-key") || url.searchParams.get("peer_public_key") || url.searchParams.get("publicKey") || url.searchParams.get("publickey");
  let privKey = url.username || url.searchParams.get("privateKey") || url.searchParams.get("private-key") || url.searchParams.get("privatekey");
  
  // جستجوی عمیق‌تر برای پارامترهایی که با حروف کوچک و بزرگ نوشته شده‌اند
  if (!pubKey || !privKey) {
    for (const [k, v] of url.searchParams.entries()) {
      const lowerK = k.toLowerCase().replace(/[-_]/g, '');
      if (!pubKey && lowerK === 'publickey') pubKey = v;
      if (!privKey && lowerK === 'privatekey') privKey = v;
    }
  }

  // تزریق خودکار کلید عمومی استاندارد Cloudflare WARP در صورت مفقود بودن
  if (!pubKey) {
    pubKey = "bmXOC+F1FxEMF9dyiK2H5/1SUtzH0JuVo51h2wPfgyo=";
  }

  const proxy = { 
    name: safeDecode(url.hash.substring(1) || url.hostname), 
    type: "wireguard", 
    server: url.hostname, 
    port: parseInt(url.port) || 51820, 
    ip: rawIp.split(",")[0].trim(), 
    "private-key": safeDecode(privKey), 
    "public-key": safeDecode(pubKey), 
    "allowed-ips": ["0.0.0.0/0"], // Mihomo requirement
    udp: true 
  };
  const reserved = url.searchParams.get("reserved"); if (reserved) proxy.reserved = reserved.split(",").map(Number);
  const mtu = url.searchParams.get("mtu"); if (mtu) proxy.mtu = parseInt(mtu);
  return proxy;
}

function parseTuic(link) {
  const url = new URL(link.replace(/^tuic:\/\//i, "http://"));
  const proxy = { 
    name: safeDecode(url.hash.substring(1) || url.hostname), 
    type: "tuic", 
    server: url.hostname, 
    port: parseInt(url.port), 
    uuid: safeDecode(url.username) || "", 
    password: safeDecode(url.password) || "",
    "disable-sni": true,         // Mihomo suggested default
    "reduce-rtt": true,          // Mihomo suggested default
    "udp-relay-mode": "native"   // Mihomo suggested default
  };
  const sni = url.searchParams.get("sni"); if (sni) proxy.sni = sni;
  const alpn = url.searchParams.get("alpn"); if (alpn) proxy.alpn = alpn.split(",");
  const congestion = url.searchParams.get("congestion_control"); if (congestion) proxy["congestion-controller"] = congestion;
  const udpRelay = url.searchParams.get("udp_relay_mode"); if (udpRelay) proxy["udp-relay-mode"] = udpRelay;
  return proxy;
}

function parseHttp(link) {
  const isHttps = link.toLowerCase().startsWith("https://"); const url = new URL(link);
  return { name: safeDecode(url.hash.substring(1) || url.hostname), type: "http", server: url.hostname, port: parseInt(url.port) || (isHttps ? 443 : 80), tls: isHttps, username: safeDecode(url.username) || "", password: safeDecode(url.password) || "" };
}

function parseSocks(link) {
  const url = new URL(link.replace(/^(socks|socks5):\/\//i, "http://"));
  return { name: safeDecode(url.hash.substring(1) || url.hostname), type: "socks5", server: url.hostname, port: parseInt(url.port) || 1080, username: safeDecode(url.username) || "", password: safeDecode(url.password) || "" };
}

function parseSnell(link) {
  const url = new URL(link.replace(/^snell:\/\//i, "http://"));
  const proxy = { name: safeDecode(url.hash.substring(1) || url.hostname), type: "snell", server: url.hostname, port: parseInt(url.port), psk: safeDecode(url.username || url.searchParams.get("psk")) || "", version: url.searchParams.get("version") || "2" };
  const obfs = url.searchParams.get("obfs"); if (obfs && obfs !== "none") { proxy["obfs-opts"] = { mode: obfs }; const host = url.searchParams.get("obfs-host"); if (host) proxy["obfs-opts"].host = host; }
  return proxy;
}

function parseSSH(link) {
  const url = new URL(link.replace(/^ssh:\/\//i, "http://"));
  return { 
    name: safeDecode(url.hash.substring(1) || url.hostname), 
    type: "ssh", 
    server: url.hostname, 
    port: parseInt(url.port) || 22, 
    user: safeDecode(url.username) || "", 
    username: safeDecode(url.username) || "", 
    password: safeDecode(url.password) || "" 
  };
}

/* ---------- Strict Mihomo Validation ---------- */

function valid(p) {
  if (!p.server || !p.port || isNaN(p.port) || p.port < 1 || p.port > 65535) return false;
  const blockedServers = ["127.0.0.1", "0.0.0.0", "localhost", "t.me", "github.com", "raw.githubusercontent.com", "google.com"];
  if (blockedServers.some(s => p.server.toLowerCase().includes(s))) return false;

  if (p["reality-opts"]) {
    const pbk = p["reality-opts"]["public-key"];
    if (!pbk || typeof pbk !== 'string') return false;
    const cleanPbk = pbk.replace(/=/g, "").trim();
    if (cleanPbk.length !== 43 || !/^[A-Za-z0-9\-_]+$/.test(cleanPbk)) return false;
    const sid = p["reality-opts"]["short-id"];
    if (sid && (typeof sid !== 'string' || !/^[0-9a-fA-F]+$/.test(sid) || sid.length % 2 !== 0 || sid.length > 16)) return false;
  }

  // Full exhaustive list of Mihomo supported ciphers based on official docs
  const validSSCiphers = [
    "aes-128-ctr", "aes-192-ctr", "aes-256-ctr",
    "aes-128-cfb", "aes-192-cfb", "aes-256-cfb",
    "aes-128-gcm", "aes-192-gcm", "aes-256-gcm",
    "aes-128-ccm", "aes-192-ccm", "aes-256-ccm",
    "aes-128-gcm-siv", "aes-256-gcm-siv",
    "chacha20-ietf", "chacha20", "xchacha20",
    "chacha20-ietf-poly1305", "xchacha20-ietf-poly1305",
    "chacha8-ietf-poly1305", "xchacha8-ietf-poly1305",
    "2022-blake3-aes-128-gcm", "2022-blake3-aes-256-gcm", "2022-blake3-chacha20-poly1305",
    "lea-128-gcm", "lea-192-gcm", "lea-256-gcm",
    "rabbit128-poly1305", "aegis-128l", "aegis-256", "aez-384",
    "deoxys-ii-256-128", "rc4-md5", "none"
  ];
  
  switch (p.type) {
    case "vmess": 
    case "vless": 
      if (!p.uuid) return false; break;
    case "trojan": 
      if (!p.password) return false; break;
    case "wireguard": 
      if (!p["private-key"]) return false; break;
    case "hysteria2": 
      if (!p.password) return false; break;
    case "hysteria": 
      if (!p.auth_str) return false; break;
    case "tuic": 
      if (!p.uuid || !p.password) return false; break;
    case "snell": 
      if (!p.psk) return false; break;
    case "ssh":
      if (!(p.password || p["private-key"]) || !(p.user || p.username)) return false; break;
    case "ss":
      if (!p.cipher || !p.password || !validSSCiphers.includes(p.cipher.toLowerCase())) return false; 
      if (p.cipher.toLowerCase().startsWith("2022-")) {
        try {
          let fixedKeys = [];
          for (const k of p.password.split(":")) {
            let cleanK = k.trim().replace(/-/g, "+").replace(/_/g, "/");
            const pad = cleanK.length % 4; if (pad === 1) return false;
            if (pad === 2) cleanK += "=="; if (pad === 3) cleanK += "=";
            if (!/^[A-Za-z0-9+/]+={0,2}$/.test(cleanK)) return false;
            atob(cleanK); fixedKeys.push(cleanK);
          }
          p.password = fixedKeys.join(":");
        } catch { return false; }
      }
      break;
    case "ssr":
      if (!p.cipher || !p.password || !p.protocol || !p.obfs || !validSSCiphers.includes(p.cipher.toLowerCase())) return false; 
      break;
  }
  return true;
}

function buildProvider(proxies) { 
  if (proxies.length === 0) return "proxies: []\n";
  return "proxies:\n" + proxies.map(p => "  - " + JSON.stringify(p)).join("\n") + "\n"; 
}

// تابع ساخت کانفیگ کامل
function buildFullConfig(proxies) {
  if (proxies.length === 0) return "proxies: []\n";
  
  const proxyListYaml = proxies.map(p => "  - " + JSON.stringify(p)).join("\n");
  const proxyNamesList = proxies.map(p => `      - "${p.name}"`).join("\n");
  
  return `global-client-fingerprint: chrome
port: 7890
socks-port: 7891
redir-port: 7892
mixed-port: 7893
tproxy-port: 7894
allow-lan: true
tcp-concurrent: true
enable-process: true
find-process-mode: always
ipv6: true
log-level: debug
geo-auto-update: true
geo-update-interval: 168
secret: ''
bind-address: '*'
unified-delay: false
disable-keep-alive: false
keep-alive-idle: 30
keep-alive-interval: 30
profile:
  store-selected: true
  store-fake-ip: true
dns:
  enable: true
  ipv6: true
  respect-rules: false
  prefer-h3: true
  cache-algorithm: arc
  use-system-hosts: true
  use-host: true
  listen: 0.0.0.0:53
  enhanced-mode: fake-ip
  fake-ip-filter-mode: blacklist
  fake-ip-range: 198.18.0.1/16
  fake-ip-filter:
    - '*.lan'
    - '*.localdomain'
    - '*.invalid'
    - '*.localhost'
    - '*.test'
    - '*.local'
    - '*.home.arpa'
    - 'time.*.com'
    - 'ntp.*.com'
    - '*.ir'
 
  default-nameserver: 
    - 8.8.8.8 
    - 8.8.4.4 
    - 1.0.0.1 
    - 1.1.1.1 
    - 9.9.9.9 
    - 9.9.9.11 
    - 9.9.9.10 
    - 94.140.14.15 
    - 94.140.15.15 
    - 223.5.5.5 
    - 77.88.8.8
  nameserver:
    - 'https://sky.rethinkdns.com/1:-J8AGH8C7_2-___f3_vZ3f_z-f9KagBI'
    - 'tls://1-7cpqagd7alx73px777p5766z3x77h6p7jjvaasa.max.rethinkdns.com'
  direct-nameserver:
    - '78.157.42.100'
    - '78.157.42.101' 
  proxy-server-nameserver: 
    - '2606:4700:4700::1111' 
    - '2606:4700:4700::1001' 
    - '2001:4860:4860::8888' 
    - '2001:4860:4860::8844' 
    - '1.1.1.1' 
    - '8.8.8.8' 
    - '8.8.4.4' 
    - '9.9.9.9' 
    - '223.5.5.5' 
    - '77.88.8.8' 
    - '2400:3200::1' 
    - '2a02:6b8::feed:0ff' 
    - '2620:fe::fe' 

sniffer:
  enable: true
  force-dns-mapping: true
  parse-pure-ip: true
  override-destination: false
  sniff:
    HTTP:
      ports: [80, 8080, 8880, 2052, 2082, 2086, 2095]
    TLS:
      ports: [443, 8443, 2053, 2083, 2087, 2096]

tun:
  enable: true
  stack: mixed
  auto-route: true
  auto-detect-interface: true
  auto-redir: true
  dns-hijack:
    - "any:53"
    - "tcp://any:53"

rule-providers:
  local_ips:
    type: http
    behavior: ipcidr
    url: https://raw.githubusercontent.com/10ium/V2rayDomains2Clash/generated/local-ips.yaml
    interval: 86400
    path: ./ruleset/local_ips.yaml
  category_ir:
    type: http
    behavior: domain
    url: https://raw.githubusercontent.com/10ium/V2rayDomains2Clash/generated/category-ir.yaml
    interval: 86400
    path: ./ruleset/category_ir.yaml
  iran:
    type: http
    behavior: classical
    url: https://raw.githubusercontent.com/10ium/clash_rules/main/iran.yaml
    interval: 86400
    path: ./ruleset/iran.yaml
  ir:
    type: http
    format: yaml
    behavior: domain
    url: "https://github.com/chocolate4u/Iran-clash-rules/releases/latest/download/ir.yaml"
    path: ./ruleset/ir.yaml
    interval: 86400
  apps:
    type: http
    format: yaml
    behavior: classical
    url: "https://github.com/chocolate4u/Iran-clash-rules/releases/latest/download/apps.yaml"
    path: ./ruleset/apps.yaml
    interval: 86400
  ircidr:
    type: http
    format: yaml
    behavior: ipcidr
    url: "https://github.com/chocolate4u/Iran-clash-rules/releases/latest/download/ircidr.yaml"
    path: ./ruleset/ircidr.yaml
    interval: 86400
  irasn:
    type: http
    format: yaml
    behavior: classical
    url: "https://raw.githubusercontent.com/Chocolate4U/Iran-clash-rules/release/irasn.yaml"
    path: ./ruleset/irasn.yaml
    interval: 86400
  arvancloud:
    type: http
    format: yaml
    behavior: ipcidr
    url: "https://raw.githubusercontent.com/Chocolate4U/Iran-clash-rules/release/arvancloud.yaml"
    path: ./ruleset/arvancloud.yaml
    interval: 86400
  derakcloud:
    type: http
    format: yaml
    behavior: ipcidr
    url: "https://raw.githubusercontent.com/Chocolate4U/Iran-clash-rules/release/derakcloud.yaml"
    path: ./ruleset/derakcloud.yaml
    interval: 86400
  iranserver:
    type: http
    format: yaml
    behavior: ipcidr
    url: "https://raw.githubusercontent.com/Chocolate4U/Iran-clash-rules/release/iranserver.yaml"
    path: ./ruleset/iranserver.yaml
    interval: 86400
  parspack:
    type: http
    format: yaml
    behavior: ipcidr
    url: "https://raw.githubusercontent.com/Chocolate4U/Iran-clash-rules/release/parspack.yaml"
    path: ./ruleset/parspack.yaml
    interval: 86400

proxies:
${proxyListYaml}

proxy-groups:
  - name: "نوع انتخاب پروکسی 🔀"
    icon: https://www.svgrepo.com/show/412721/choose.svg
    type: select
    proxies:
      - "خودکار (بهترین پینگ) 🤖"
      - "دستی 🤏🏻"
      - "بدون فیلترشکن 🛡️"
      - "قطع اینترنت ⛔"
  - name: "دستی 🤏🏻"
    type: select
    icon: https://www.svgrepo.com/show/372331/cursor-hand-click.svg
    proxies:
${proxyNamesList}
  - name: "خودکار (بهترین پینگ) 🤖"
    type: url-test
    icon: https://www.svgrepo.com/show/7876/speedometer.svg
    url: https://api.v2fly.org/checkConnection.svgz
    interval: 300
    tolerance: 50
    lazy: true
    proxies:
${proxyNamesList}
  - name: سایتای ایرانی 🇮🇷
    type: select
    icon: https://upload.wikimedia.org/wikipedia/commons/3/36/Flag_of_Iran_%28civil%29.svg
    proxies:
      - "بدون فیلترشکن 🛡️"
      - "نوع انتخاب پروکسی 🔀"
      - "خودکار (بهترین پینگ) 🤖"
      - "دستی 🤏🏻"
      - "اجازه ندادن 🚫"
  - name: تلگرام 💬
    type: select
    icon: https://www.svgrepo.com/show/354443/telegram.svg
    proxies:
      - "نوع انتخاب پروکسی 🔀"
      - "بدون فیلترشکن 🛡️"
      - "خودکار (بهترین پینگ) 🤖"
      - "دستی 🤏🏻"
      - "اجازه ندادن 🚫"
  - name: "بدون فیلترشکن 🛡️"
    type: select
    icon: https://www.svgrepo.com/show/6318/connection.svg
    proxies:
      - DIRECT
    hidden: true
  - name: "قطع اینترنت ⛔"
    type: select
    icon: https://www.svgrepo.com/show/305372/wifi-off.svg
    proxies:
      - REJECT
    hidden: true
  - name: "اجازه ندادن 🚫"
    type: select
    icon: https://www.svgrepo.com/show/444307/gui-ban.svg
    proxies:
      - REJECT
    hidden: true

rules:
  - RULE-SET,local_ips,بدون فیلترشکن 🛡️
  - PROCESS-NAME,Telegram.exe,تلگرام 💬
  - PROCESS-NAME,org.telegram.messenger,تلگرام 💬
  - PROCESS-NAME,org.telegram.messenger.web,تلگرام 💬
  - IP-CIDR,10.10.34.0/24,نوع انتخاب پروکسی 🔀
  - RULE-SET,apps,سایتای ایرانی 🇮🇷
  - RULE-SET,iran,سایتای ایرانی 🇮🇷
  - RULE-SET,arvancloud,سایتای ایرانی 🇮🇷
  - RULE-SET,derakcloud,سایتای ایرانی 🇮🇷
  - RULE-SET,iranserver,سایتای ایرانی 🇮🇷
  - RULE-SET,parspack,سایتای ایرانی 🇮🇷
  - RULE-SET,irasn,سایتای ایرانی 🇮🇷
  - RULE-SET,ircidr,سایتای ایرانی 🇮🇷
  - RULE-SET,ir,سایتای ایرانی 🇮🇷
  - RULE-SET,category_ir,سایتای ایرانی 🇮🇷
  - MATCH,نوع انتخاب پروکسی 🔀

ntp:
  enable: true
  server: "time.apple.com"
  port: 123
  interval: 30
`;
}

/* ==========================================
   رابط کاربری حرفه‌ای 
   ========================================== */

function getHTML(origin) {
  const protocols = ["ssh", "hy2", "anytls", "snell", "ssr", "vless", "vmess", "ss", "trojan", "wg", "hy", "tuic", "socks", "http"];
  
  return `
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="icon" href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22><text y=%22.9em%22 font-size=%2290%22>⚡</text></svg>">
    <title>Mihomo Manager | مدیریت پیشرفته اشتراک</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Vazirmatn:wght@300;400;700&display=swap');
        :root {
            --primary: #8e44ad; --secondary: #3498db; --accent: #1abc9c; --danger: #e74c3c;
            --bg: #0b0e14; --glass: rgba(255, 255, 255, 0.03); --border: rgba(255, 255, 255, 0.08);
        }
        * { box-sizing: border-box; scroll-behavior: smooth; }
        body {
            font-family: 'Vazirmatn', sans-serif; background: radial-gradient(circle at top right, #1a1a2e, #0b0e14);
            color: #ecf0f1; margin: 0; min-height: 100vh; display: flex; justify-content: center; align-items: flex-start; padding: 40px 20px;
        }
        .app-card {
            width: 100%; max-width: 950px; background: var(--glass); backdrop-filter: blur(20px);
            border: 1px solid var(--border); border-radius: 30px; padding: 40px; box-shadow: 0 40px 100px rgba(0,0,0,0.5);
        }
        h1 { text-align: center; font-weight: 700; font-size: 2.2rem; margin-bottom: 30px; letter-spacing: -1px; background: linear-gradient(90deg, #fff, #8e44ad); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .subtitle { text-align: center; color: #95a5a6; font-size: 0.9rem; margin-top: -20px; margin-bottom: 40px; }

        .section-label { display: block; font-size: 1rem; font-weight: 700; color: var(--primary); margin-bottom: 12px; margin-top: 35px; border-right: 3px solid var(--primary); padding-right: 10px; }
        
        textarea, input {
            width: 100%; background: rgba(0,0,0,0.4); border: 1px solid var(--border); border-radius: 15px;
            color: #fff; padding: 15px; font-size: 0.95rem; transition: 0.3s; font-family: 'Vazirmatn', sans-serif;
        }
        textarea:focus, input:focus { border-color: var(--primary); outline: none; box-shadow: 0 0 15px rgba(142, 68, 173, 0.3); }
        textarea { height: 160px; font-family: monospace; direction: ltr; }

        .config-table { width: 100%; margin-top: 15px; border-collapse: separate; border-spacing: 0 8px; }
        .config-table th { color: #bdc3c7; font-size: 0.85rem; text-transform: uppercase; text-align: center; padding-bottom: 10px; border-bottom: 1px solid var(--border); }
        .config-table td { background: rgba(0,0,0,0.2); padding: 10px; text-align: center; transition: all 0.2s; }
        .config-table tr { transition: transform 0.2s, box-shadow 0.2s; }
        .config-table tr.dragging { opacity: 0.5; background: rgba(142, 68, 173, 0.2); }
        .config-table tr.drag-over { border-top: 2px solid var(--accent); transform: scale(1.01); }
        
        .config-table td:first-child { border-radius: 0 12px 12px 0; font-weight: bold; color: var(--accent); display: flex; align-items: center; justify-content: space-between; gap: 10px; }
        .config-table td:last-child { border-radius: 12px 0 0 12px; width: 150px; }
        .config-table input { width: 100%; padding: 8px; text-align: center; border-radius: 8px; font-weight: bold; font-size: 0.9rem; }
        
        .drag-handle { cursor: grab; color: #7f8c8d; font-size: 1.2rem; padding: 0 5px; user-select: none; }
        .drag-handle:active { cursor: grabbing; }
        
        .move-btns { display: flex; flex-direction: column; gap: 2px; }
        .move-btn { background: transparent; border: none; color: #95a5a6; cursor: pointer; padding: 2px; font-size: 0.7rem; }
        .move-btn:hover { color: #fff; }

        .help-text { font-size: 0.75rem; color: #7f8c8d; display: block; margin-top: 5px; }

        .btn-generate {
            width: 100%; padding: 20px; border: none; border-radius: 15px; cursor: pointer;
            font-weight: 700; font-size: 1.2rem; margin-top: 40px; transition: 0.3s;
            background: linear-gradient(135deg, var(--primary), var(--secondary)); color: #fff;
            box-shadow: 0 10px 20px rgba(0,0,0,0.2);
        }
        .btn-generate:hover { transform: translateY(-3px); box-shadow: 0 15px 30px rgba(142, 68, 173, 0.5); }
        .btn-generate:disabled { background: #7f8c8d; cursor: not-allowed; transform: none; box-shadow: none; }

        .stats-panel {
            margin-top: 20px; padding: 20px; background: rgba(0,0,0,0.2); border-radius: 20px;
            display: grid; grid-template-columns: repeat(auto-fill, minmax(110px, 1fr)); gap: 15px; border: 1px solid var(--border);
        }
        .stat-badge { background: rgba(255,255,255,0.05); padding: 12px; border-radius: 12px; text-align: center; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
        .stat-badge div:first-child { font-size: 0.75rem; color: #7f8c8d; }
        .stat-badge div:last-child { font-weight: 700; color: var(--accent); font-size: 1.2rem; margin-top: 5px; }

        .errors-panel { display: none; margin-top: 20px; background: rgba(231, 76, 60, 0.1); border: 1px solid rgba(231, 76, 60, 0.3); border-radius: 15px; padding: 20px; }
        .errors-panel h3 { color: var(--danger); font-size: 1rem; margin-top: 0; margin-bottom: 10px; }
        .error-item { font-size: 0.85rem; color: #ff7979; margin-bottom: 5px; direction: ltr; text-align: left; background: rgba(0,0,0,0.3); padding: 8px; border-radius: 6px; word-break: break-all;}

        .result-area { margin-top: 40px; padding: 30px; background: rgba(255,255,255,0.02); border-radius: 20px; border: 1px solid var(--border); display: none; }
        .result-area input { font-size: 1.1rem; color: var(--accent); background: rgba(0,0,0,0.5); }
        .action-group { display: grid; grid-template-columns: repeat(3, 1fr); gap: 15px; margin-top: 20px; }
        .btn-action {
            padding: 15px; border: 1px solid var(--border); border-radius: 12px; background: var(--glass);
            color: #fff; cursor: pointer; font-size: 1rem; font-weight: bold; transition: 0.2s; font-family: 'Vazirmatn';
            display: flex; justify-content: center; align-items: center; gap: 8px;
        }
        .btn-action:hover { background: #fff; color: #000; }
        .success-copy { color: var(--accent); font-size: 0.9rem; text-align: center; margin-top: 15px; display: none; font-weight: bold; }
        
        .loader { border: 3px solid rgba(255,255,255,0.1); border-top: 3px solid var(--accent); border-radius: 50%; width: 24px; height: 24px; animation: spin 1s linear infinite; display: inline-block; vertical-align: middle; margin-right: 10px; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body>
    <div class="app-card">
        <h1>Mihomo Manager</h1>
        <div class="subtitle">موتور تجمیع فوق سریع، اعتبارسنجی و اولویت‌بندی اشتراک‌ها</div>

        <label class="section-label">سورس‌های اختصاصی شما</label>
        <span class="help-text" style="margin-bottom:10px;">هر خط یک لینک وارد کنید. محتوای لینک‌ها می‌تواند متن ساده یا Base64 باشد.</span>
        <textarea id="subs" placeholder="https://example.com/sub1.txt&#10;https://example.com/sub2.txt"></textarea>

        <label class="section-label">تنظیمات خروجی نهایی</label>
        <div style="margin-bottom: 20px;">
            <label style="font-size: 0.9rem; color: #bdc3c7; display: block; margin-bottom: 8px;">نوع خروجی (فرمت فایل)</label>
            <select id="output_type" style="width: 100%; background: rgba(0,0,0,0.4); border: 1px solid var(--border); border-radius: 15px; color: #fff; padding: 15px; font-size: 0.95rem; margin-bottom: 15px; font-family: 'Vazirmatn', sans-serif; cursor: pointer;">
                <option value="provider" style="background: #0b0e14;">پروکسی پروایدر (Proxy Provider) - پیش‌فرض</option>
                <option value="config" style="background: #0b0e14;">کانفیگ کامل کلش (Full Config)</option>
            </select>

            <label style="font-size: 0.9rem; color: #bdc3c7; display: block; margin-bottom: 8px;">حداکثر کل کانفیگ‌های خروجی</label>
            <input type="number" id="limit_total" placeholder="نامحدود (خالی بگذارید)">
        </div>

        <!-- ویژگی جدید: Amnezia WireGuard Options -->
        <label class="section-label">تنظیمات پیشرفته WireGuard (Amnezia)</label>
        <div style="margin-bottom: 20px; background: rgba(0,0,0,0.2); border: 1px solid var(--border); border-radius: 15px; padding: 15px;">
            <label style="display: flex; align-items: center; cursor: pointer; color: #ecf0f1; font-size: 0.95rem; font-weight: bold;">
                <input type="checkbox" id="enable_awg" onchange="toggleAwgOptions()" style="width: 20px; height: 20px; margin-left: 10px; cursor: pointer; box-shadow: none;">
                فعال‌سازی قابلیت Amnezia-WG برای کانفیگ‌های وایرگارد
            </label>
            <span class="help-text" style="margin-right: 30px; margin-top: 5px;">با فعال‌سازی این گزینه، پارامترهای ضد فیلتر فقط روی کانفیگ‌های WireGuard اعمال می‌شود. بقیه پروکسی‌ها دست‌نخورده می‌مانند.</span>
            
            <div id="awg_options_container" style="display: none; margin-top: 20px; border-top: 1px solid rgba(255,255,255,0.05); padding-top: 15px;">
                <label style="font-size: 0.9rem; color: #bdc3c7; display: block; margin-bottom: 8px;">انتخاب پروفایل پارامترها</label>
                <select id="awg_profile" onchange="applyAwgProfile()" style="width: 100%; background: rgba(0,0,0,0.4); border: 1px solid var(--border); border-radius: 10px; color: #fff; padding: 10px; margin-bottom: 15px; font-family: 'Vazirmatn', sans-serif; cursor: pointer;">
                    <!-- مقادیر توسط جاوااسکریپت پر می‌شوند -->
                </select>
                
                <div id="awg_custom_inputs" style="display: grid; grid-template-columns: repeat(3, 1fr); gap: 15px; margin-top: 10px;">
                    <div>
                        <label style="font-size:0.85rem; color:#bdc3c7; display:block; margin-bottom:5px;">Jc (Junk Count)</label>
                        <input type="number" id="awg_jc" placeholder="مثال: 4" value="4">
                    </div>
                    <div>
                        <label style="font-size:0.85rem; color:#bdc3c7; display:block; margin-bottom:5px;">Jmin (Junk Min)</label>
                        <input type="number" id="awg_jmin" placeholder="مثال: 40" value="40">
                    </div>
                    <div>
                        <label style="font-size:0.85rem; color:#bdc3c7; display:block; margin-bottom:5px;">Jmax (Junk Max)</label>
                        <input type="number" id="awg_jmax" placeholder="مثال: 100" value="100">
                    </div>
                </div>
                <span class="help-text" style="margin-top: 15px;">مقادیر ثابت s1:0, s2:0, h1:1, h2:2, h3:3, h4:4 به‌صورت خودکار پس از ساخت خروجی اعمال خواهند شد.</span>
            </div>
        </div>

        <label class="section-label">اولویت‌بندی و فیلتر پروتکل‌ها</label>
        <span class="help-text" style="margin-bottom:10px;">برای تغییر اولویت، سطرها را با ماوس بکشید (Drag & Drop) یا از فلش‌ها استفاده کنید. عدد ۰ باعث حذف کامل آن پروتکل می‌شود.</span>
        
        <table class="config-table">
            <thead>
                <tr>
                    <th style="text-align: right; padding-right: 35px;">اولویت / پروتکل</th>
                    <th>محدودیت تعداد (۰ = حذف)</th>
                </tr>
            </thead>
            <tbody id="protocol_table_body">
                ${protocols.map((p) => `
                    <tr draggable="true" data-protocol="${p}" class="draggable-row">
                        <td>
                            <div style="display:flex; align-items:center; gap:10px;">
                                <span class="drag-handle" title="بکشید و رها کنید">☰</span>
                                <div class="move-btns">
                                    <button class="move-btn" onclick="moveUp(this)" title="انتقال به بالا">▲</button>
                                    <button class="move-btn" onclick="moveDown(this)" title="انتقال به پایین">▼</button>
                                </div>
                                <span style="margin-right:10px;">${p.toUpperCase()}</span>
                            </div>
                        </td>
                        <td><input type="number" class="limit-input" placeholder="∞"></td>
                    </tr>
                `).join('')}
            </tbody>
        </table>

        <button id="btn_submit" class="btn-generate" onclick="process()">🚀 بررسی و تولید لینک سابسکریپشن</button>

        <div id="errors_panel" class="errors-panel"></div>

        <div id="stats_container" style="display:none">
            <label class="section-label">آمار پردازش (پروکسی‌های سالم و فیلتر شده)</label>
            <div id="stats_panel" class="stats-panel"></div>
        </div>

        <div class="result-area" id="result_area">
            <label class="section-label" style="margin-top:0;">🔗 لینک نهایی سابسکریپشن شما</label>
            <input type="text" id="final_url" readonly>
            <div id="copy_msg" class="success-copy">✅ لینک با موفقیت کپی شد!</div>
            <div class="action-group">
                <button class="btn-action" onclick="copyUrl()">📋 کپی لینک</button>
                <button class="btn-action" onclick="window.open(document.getElementById('final_url').value, '_blank')">🌍 بازکردن</button>
                <button class="btn-action" onclick="downloadYaml()">💾 دانلود فایل</button>
            </div>
        </div>
    </div>

    <script>
        const origin = "${origin}";

        // --- Amnezia WG Profiles Definition ---
        const AMNEZIA_PROFILES = [
            { id: 'custom',                          name: 'سفارشی (مقادیر دستی)', isCustom: true },
            { id: 'Optimal Balanced (Daily Use)',    name: 'Optimal (متعادل)',                jc: 4,  jmin: 64,  jmax: 120 },
            { id: 'Weak_Net (low-bandwidth)',        name: 'Weak_Net (اینترنت ضعیف)',         jc: 6,  jmin: 64,  jmax: 80  },
            { id: 'Aggressive Pattern',              name: 'Aggressive (تهاجمی)',             jc: 8,  jmin: 64,  jmax: 150 },
            { id: 'Fast (low-handshake-overhead)',   name: 'Fast (سریع)',                     jc: 2,  jmin: 64,  jmax: 70  },
            { id: 'Proton (minimal compatibility)',  name: 'Proton (سازگاری پروتون)',         jc: 4,  jmin: 40,  jmax: 70  },
            { id: 'bpb (legacy / balanced)',         name: 'BPB (کلاسیک)',                    jc: 5,  jmin: 50,  jmax: 100 },
            { id: 'hamedp71 (compatibility)',        name: 'HamedP71 (سازگار بالا)',          jc: 4,  jmin: 40,  jmax: 250 },
            { id: 'Rus_Micro (Micro-Noise)',         name: 'Rus_Micro (نویز کم)',             jc: 3,  jmin: 10,  jmax: 30  },
            { id: 'Rus_Flood (Heavy Flood)',         name: 'Rus_Flood (ضد DPI قوی)',          jc: 10, jmin: 30,  jmax: 60  },
            { id: 'Stalinium Strategy (Maximum)',    name: 'Stalinium (حداکثر مقاومت)',       jc: 31, jmin: 20,  jmax: 40  },
            { id: 'High-Entropy Obfuscation',        name: 'High Entropy (رمزنگاری سنگین)',   jc: 33, jmin: 132, jmax: 1200},
            { id: 'Heavy Traffic Simulation',        name: 'Heavy Traffic (شبیه‌ساز ترافیک)', jc: 50, jmin: 5,   jmax: 1500},
            { id: 'MetaCubeX Fixed Window',          name: 'MetaCubeX (مخصوص کلاینت)',        jc: 5,  jmin: 500, jmax: 501 },
            { id: 'Amnezia Official Default',        name: 'Amnezia Default (پیش‌فرض)',       jc: 8,  jmin: 50,  jmax: 1000},
            { id: 'Gaming / Ultra-Low Overhead',     name: 'Gaming (مخصوص بازی / پینگ کم)',   jc: 3,  jmin: 64,  jmax: 80  }
        ];

        function initAwgProfiles() {
            const select = document.getElementById('awg_profile');
            AMNEZIA_PROFILES.forEach(p => {
                const opt = document.createElement('option');
                opt.value = p.id;
                opt.textContent = p.name;
                opt.style.background = '#0b0e14';
                select.appendChild(opt);
            });
            // ست کردن یک پروفایل پیشنهادی پیش‌فرض
            select.value = 'Optimal Balanced (Daily Use)';
            applyAwgProfile();
        }

        function toggleAwgOptions() {
            const container = document.getElementById('awg_options_container');
            const isEnabled = document.getElementById('enable_awg').checked;
            container.style.display = isEnabled ? 'block' : 'none';
        }

        function applyAwgProfile() {
            const selectedId = document.getElementById('awg_profile').value;
            const customInputs = document.getElementById('awg_custom_inputs');
            
            if (selectedId === 'custom') {
                customInputs.style.display = 'grid';
            } else {
                // پنهان کردن فیلدهای دستی و تنظیم مقادیر از پیش‌فرض
                customInputs.style.display = 'none';
                const profile = AMNEZIA_PROFILES.find(p => p.id === selectedId);
                if (profile) {
                    document.getElementById('awg_jc').value = profile.jc;
                    document.getElementById('awg_jmin').value = profile.jmin;
                    document.getElementById('awg_jmax').value = profile.jmax;
                }
            }
        }
        
        // اجرای مقداردهی اولیه لیست پروفایل‌ها
        initAwgProfiles();

        // --- Drag & Drop ---
        let draggedItem = null;
        document.querySelectorAll('.draggable-row').forEach(row => {
            row.addEventListener('dragstart', function(e) { draggedItem = this; setTimeout(() => this.classList.add('dragging'), 0); });
            row.addEventListener('dragend', function() { setTimeout(() => { this.classList.remove('dragging'); draggedItem = null; }, 0); });
            row.addEventListener('dragover', function(e) { e.preventDefault(); });
            row.addEventListener('dragenter', function(e) { e.preventDefault(); if(this !== draggedItem) this.classList.add('drag-over'); });
            row.addEventListener('dragleave', function() { this.classList.remove('drag-over'); });
            row.addEventListener('drop', function(e) {
                this.classList.remove('drag-over');
                if (this !== draggedItem) {
                    let tbody = this.parentNode; let rows = Array.from(tbody.children);
                    let draggedIndex = rows.indexOf(draggedItem); let targetIndex = rows.indexOf(this);
                    if (draggedIndex < targetIndex) this.after(draggedItem); else this.before(draggedItem);
                }
            });
        });

        function moveUp(btn) { const row = btn.closest('tr'); if (row.previousElementSibling) row.parentNode.insertBefore(row, row.previousElementSibling); }
        function moveDown(btn) { const row = btn.closest('tr'); if (row.nextElementSibling) row.parentNode.insertBefore(row.nextElementSibling, row); }

        async function process() {
            const btn = document.getElementById('btn_submit');
            btn.disabled = true;
            btn.innerHTML = '<span class="loader"></span> در حال پردازش سورس‌ها (سرعت بالا)...';
            
            const errPanel = document.getElementById('errors_panel');
            errPanel.style.display = 'none'; errPanel.innerHTML = '';

            const statsContainer = document.getElementById('stats_container');
            const statsDiv = document.getElementById('stats_panel');
            statsContainer.style.display = 'block';
            statsDiv.innerHTML = '<div style="grid-column: 1/-1; text-align:center; color:#bdc3c7;">در حال واکشی و اعتبارسنجی سریع... لطفاً صبور باشید.</div>';
            document.getElementById('result_area').style.display = 'none';

            const query = new URLSearchParams();
            const subsInput = document.getElementById('subs').value.trim();
            if (subsInput) {
                const subsArray = subsInput.split('\\n').map(s => s.trim()).filter(Boolean);
                query.set('subs', subsArray.join(','));
            }

            const limitTotal = document.getElementById('limit_total').value;
            if (limitTotal) query.set('limit_total', limitTotal);

            // دریافت نوع خروجی انتخابی کاربر
            const outputType = document.getElementById('output_type').value;
            query.set('output', outputType);

            // ارسال مقادیر Amnezia WG در صورت فعال بودن
            if (document.getElementById('enable_awg').checked) {
                query.set('awg', '1');
                query.set('awg_jc', document.getElementById('awg_jc').value);
                query.set('awg_jmin', document.getElementById('awg_jmin').value);
                query.set('awg_jmax', document.getElementById('awg_jmax').value);
            }

            const rows = document.querySelectorAll('#protocol_table_body tr');
            rows.forEach((row, index) => {
                const protocol = row.dataset.protocol;
                query.set('prio_' + protocol, index + 1);
                const lim = row.querySelector('.limit-input').value;
                if (lim !== "") query.set('limit_' + protocol, lim);
            });

            try {
                const statsQuery = new URLSearchParams(query);
                statsQuery.set('format', 'json');
                const res = await fetch(origin + "/mihomo?" + statsQuery.toString());
                if (!res.ok) throw new Error("ارتباط با سرور برقرار نشد.");
                
                const data = await res.json();

                if (data.errors && data.errors.length > 0) {
                    errPanel.style.display = 'block';
                    let errHtml = '<h3>⚠️ خطاهای واکشی سورس‌ها</h3>';
                    data.errors.forEach(err => {
                        errHtml += '<div class="error-item"><b>URL:</b> ' + err.url + '<br><b>Error:</b> ' + err.reason + '</div>';
                    });
                    errPanel.innerHTML = errHtml;
                }

                let html = '<div class="stat-badge" style="grid-column: 1/-1; border: 1px solid var(--accent)"><div>تعداد کل خروجی</div><div>' + data.total + '</div></div>';
                if (data.total === 0) {
                    html += '<div style="grid-column: 1/-1; text-align:center; color:var(--danger); margin-top:10px;">هیچ پروکسی سالمی یافت نشد یا همه فیلتر شده‌اند!</div>';
                } else {
                    for (const [type, count] of Object.entries(data.types)) {
                        html += '<div class="stat-badge"><div>' + type.toUpperCase() + '</div><div>' + count + '</div></div>';
                    }
                }
                statsDiv.innerHTML = html;

                if (data.total > 0) {
                    const finalUrl = origin + "/mihomo?" + query.toString();
                    document.getElementById('final_url').value = finalUrl;
                    document.getElementById('result_area').style.display = 'block';
                }

            } catch (e) {
                statsDiv.innerHTML = '<div style="grid-column: 1/-1; text-align:center; color:var(--danger);">خطا در پردازش اطلاعات.</div>';
                console.error(e);
            } finally {
                btn.disabled = false;
                btn.innerHTML = '🚀 بررسی و تولید لینک سابسکریپشن';
                window.scrollTo({ top: document.body.scrollHeight, behavior: 'smooth' });
            }
        }

        function copyUrl() {
            const copyText = document.getElementById("final_url");
            copyText.select(); document.execCommand("copy");
            const msg = document.getElementById('copy_msg');
            msg.style.display = 'block'; setTimeout(() => msg.style.display = 'none', 3000);
        }

        async function downloadYaml() {
            const url = document.getElementById('final_url').value;
            try {
                const res = await fetch(url);
                const content = await res.text();
                const blob = new Blob([content], { type: 'text/yaml' });
                const a = document.createElement('a');
                a.href = URL.createObjectURL(blob);
                a.download = 'mihomo_custom_sub.yaml';
                document.body.appendChild(a); a.click(); document.body.removeChild(a); URL.revokeObjectURL(a.href);
            } catch(e) { alert("خطا در دانلود فایل!"); }
        }
    </script>
</body>
</html>
  `;
}
