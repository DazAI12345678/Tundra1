<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
<title>Tundra Biome — Mobile Pre-Beta</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<style>
  html, body { margin: 0; padding: 0; overflow: hidden; background: #000; height: 100%; width: 100%;
    touch-action: none; -webkit-user-select: none; user-select: none; }
  canvas { display: block; touch-action: none; }
  * { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif; box-sizing: border-box; }

  #hud { position: fixed; inset: 0; pointer-events: none; color: #eaf6f8; }
  #crosshair {
    position: absolute; top: 50%; left: 50%; width: 6px; height: 6px;
    margin: -3px 0 0 -3px; background: rgba(255,255,255,0.7); border-radius: 50%;
  }

  #bars-panel {
    position: absolute; top: env(safe-area-inset-top, 10px); left: 10px; width: 46vw; max-width: 190px;
    background: rgba(10,16,22,0.75); border: 1px solid #33414d; border-radius: 8px; padding: 6px 8px;
  }
  .bar-label { font-size: 0.56rem; color: #a9c2d1; display:flex; justify-content: space-between; margin-bottom:2px;}
  .bar-bg { width:100%; height:8px; background:#181f26; border-radius:5px; overflow:hidden; margin-bottom: 5px; }
  #bh-fill { height:100%; width:100%; background: linear-gradient(90deg,#e03c3c,#f0a63a,#4fd18a); transition: width .3s ease; }
  #health-fill { height:100%; width:100%; background: linear-gradient(90deg,#8a2a2a,#c94a4a,#e87a7a); transition: width .3s ease; }
  #scent-badge {
    display: none; font-size: 0.54rem; color: #ffb37a; margin-top: 3px;
    background: rgba(224,120,60,0.15); border: 1px solid #a85c2a; border-radius: 5px; padding: 2px 5px;
  }

  #items-panel {
    position: absolute; top: env(safe-area-inset-top, 10px); right: 10px; width: 34vw; max-width: 140px; text-align: right;
    background: rgba(10,16,22,0.75); border: 1px solid #33414d; border-radius: 8px; padding: 6px 8px;
    font-size: 0.6rem; color: #cfe0ea;
  }

  #objectives {
    position: absolute; top: 76px; left: 10px; width: 46vw; max-width: 190px;
    background: rgba(10,16,22,0.7); border: 1px solid #33414d; border-radius: 8px; padding: 6px 8px;
    font-size: 0.54rem; color: #cfe0ea;
  }
  #objectives h4 { margin: 0 0 4px; font-size: 0.54rem; color: #8fa8b8; letter-spacing: 0.5px; }
  #objectives div { display:flex; justify-content: space-between; padding: 1px 0; }
  .obj-done { color: #4fd18a; }
  .obj-dormant { color: #6f8291; }
  .obj-carrying { color: #f0a63a; }

  #progress-wrap {
    position: absolute; bottom: 175px; left: 50%; transform: translateX(-50%);
    width: 50vw; max-width: 200px; display: none;
  }
  #progress-bg { width:100%; height:8px; background:#181f26; border-radius:5px; overflow:hidden; border:1px solid #33414d;}
  #progress-fill { height:100%; width:0%; background:#5c9ab8; }

  #roar-banner {
    position: absolute; top: env(safe-area-inset-top, 10px); left: 50%; transform: translateX(-50%);
    background: rgba(224,60,60,0.3); border: 1px solid #e03c3c; color: #ffdada;
    padding: 6px 14px; border-radius: 8px; font-size: 0.7rem; display: none; font-weight: 700;
  }

  #playdead-btn {
    position: absolute; bottom: 220px; left: 50%; transform: translateX(-50%);
    background: rgba(224,60,60,0.5); border: 2px solid #e03c3c; color: #fff;
    padding: 12px 22px; border-radius: 10px; font-size: 0.8rem; font-weight: 700;
    display: none; pointer-events: all;
  }

  #bear-status {
    position: absolute; bottom: 195px; left: 50%; transform: translateX(-50%);
    font-size: 0.58rem; color: #d9a0a0; display:none;
  }

  #joystick-zone { position: absolute; bottom: 0; left: 0; width: 42vw; height: 55vh; pointer-events: all; }
  #joystick-base {
    position: absolute; width: 84px; height: 84px; border-radius: 50%;
    background: rgba(92,154,184,0.15); border: 2px solid rgba(92,154,184,0.5);
    display: none; transform: translate(-50%,-50%);
  }
  #joystick-knob {
    position: absolute; width: 38px; height: 38px; border-radius: 50%;
    background: rgba(92,154,184,0.55); border: 1px solid rgba(207,238,245,0.6);
    top: 23px; left: 23px;
  }
  #look-zone { position: absolute; bottom: 0; right: 0; width: 58vw; height: 100vh; pointer-events: all; }

  #action-buttons {
    position: absolute; bottom: 26px; right: 12px; display: flex; flex-direction: column; gap: 9px;
    pointer-events: all; z-index: 5;
  }
  .action-btn {
    width: 62px; height: 62px; border-radius: 50%;
    background: rgba(92,154,184,0.25); border: 2px solid rgba(92,154,184,0.7);
    color: #eaf6f8; font-size: 0.54rem; font-weight: 700; text-align: center;
    display: flex; align-items: center; justify-content: center; line-height: 1.1;
  }
  .action-btn.pressed { background: rgba(92,154,184,0.55); }
  #firecracker-btn { background: rgba(224,120,60,0.25); border-color: rgba(224,120,60,0.7); }
  #firecracker-btn.pressed { background: rgba(224,120,60,0.55); }
  #bandage-btn { background: rgba(79,209,138,0.2); border-color: rgba(79,209,138,0.6); }
  #bandage-btn.pressed { background: rgba(79,209,138,0.5); }

  #start-screen, #gameover-screen {
    position: fixed; inset: 0; background: rgba(4,7,10,0.96);
    display: flex; flex-direction: column; align-items: center; justify-content: center;
    color: #eaf6f8; text-align: center; padding: 22px; pointer-events: all; z-index: 20;
  }
  #gameover-screen { display: none; }
  #start-screen h1, #gameover-screen h1 { font-size: 1.0rem; margin-bottom: 6px; }
  #start-screen p, #gameover-screen p { font-size: 0.65rem; color: #a9c2d1; max-width: 300px; margin-bottom: 16px; line-height:1.55; }
  .diff-row { display:flex; gap:6px; margin-bottom: 18px; flex-wrap: wrap; justify-content: center; }
  .diff-btn {
    padding: 8px 10px; font-size: 0.62rem; background:#141c24; border:1px solid #33414d;
    color:#a9c2d1;  function completeWood() {
    woodProgress = 0; woodFail = 0; woodDormant = true;
    const [minD, maxD] = d.dormancy;
    woodDormantTimer = minD + Math.random()*(maxD-minD);
    bh = Math.min(100, bh + 6);
    woodStage = "chop";
    setObjectiveLabel(woodEl, "done", "Chop & deliver firewood");
    playTaskComplete();
    setTimeout(() => { if (woodDormant) setObjectiveLabel(woodEl, "dormant", "Chop & deliver firewood"); }, 1400);
  }

  function breakWood() {
    woodFail = 0; woodProgress = 0; woodStage = "chop";
    bh -= 7;
    playTaskBreak();
    setObjectiveLabel(woodEl, "active", "Chop & deliver firewood");
  }

  // ---------- STATE ----------
  let bh = 100;
  let playerHealth = 100;
  let hasScent = false;
  let gameOver = false;
  let firecrackerCount = 3;
  let bandageCount = 3;
  let resourcesFound = 0;

  let bearState = "idle";
  let roarTimer = 0;
  let playDeadWindow = false;
  let playDeadSuccess = false;

  const activeFirecrackers = [];

  // ---------- TOUCH CONTROLS ----------
  let yaw = Math.PI, pitch = 0;
  let moveVecX = 0, moveVecY = 0;
  let interacting = false;

  const joystickZone = document.getElementById("joystick-zone");
  const joystickBase = document.getElementById("joystick-base");
  const joystickKnob = document.getElementById("joystick-knob");
  const lookZone = document.getElementById("look-zone");

  let joyTouchId = null, joyOriginX = 0, joyOriginY = 0;
  let lookTouchId = null, lookLastX = 0, lookLastY = 0;

  joystickZone.addEventListener("touchstart", (e) => {
    const t = e.changedTouches[0];
    joyTouchId = t.identifier;
    joyOriginX = t.clientX; joyOriginY = t.clientY;
    joystickBase.style.display = "block";
    joystickBase.style.left = joyOriginX + "px";
    joystickBase.style.top = joyOriginY + "px";
    joystickKnob.style.left = "23px"; joystickKnob.style.top = "23px";
    ensureAudio();
    e.preventDefault();
  }, {passive:false});

  joystickZone.addEventListener("touchmove", (e) => {
    for (const t of e.changedTouches) {
      if (t.identifier !== joyTouchId) continue;
      let dx = t.clientX - joyOriginX;
      let dy = t.clientY - joyOriginY;
      const maxR = 42;
      const dist = Math.min(Math.hypot(dx,dy), maxR);
      const angle = Math.atan2(dy,dx);
      dx = Math.cos(angle)*dist; dy = Math.sin(angle)*dist;
      joystickKnob.style.left = (23+dx) + "px";
      joystickKnob.style.top = (23+dy) + "px";
      moveVecX = dx / maxR;
      moveVecY = dy / maxR;
    }
    e.preventDefault();
  }, {passive:false});

  function endJoystick(e) {
    for (const t of e.changedTouches) {
      if (t.identifier !== joyTouchId) continue;
      joyTouchId = null;
      moveVecX = 0; moveVecY = 0;
      joystickBase.style.display = "none";
    }
  }
  joystickZone.addEventListener("touchend", endJoystick);
  joystickZone.addEventListener("touchcancel", endJoystick);

  lookZone.addEventListener("
