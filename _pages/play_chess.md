---
layout: single
title: "Play Chess vs My LLM"
permalink: /play-chess/
---

<link
  rel="stylesheet"
  href="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard.min.css"
/>

<div id="board" class="mx-auto my-6" style="width: 400px;"></div>
<p id="status" class="text-center font-semibold">Your turn</p>

<script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/0.10.3/chess.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard.min.js"></script>
<script>
  const hfApi = "https://hf.space/embed/USERNAME/chess-llm-gradio/+/api/predict/";
  const game = new Chess();
  const board = Chessboard("board", {
    position: game.fen(),
    draggable: true,
    onDrop: async (source, target) => {
      // 1) try the user move
      const move = game.move({ from: source, to: target, promotion: "q" });
      if (!move) return "snapback";
      updateStatus(`You played ${move.san}… thinking`);
      board.position(game.fen());

      // 2) call HF Space API
      const res = await fetch(hfApi, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ data: [ game.fen(), move.uci ] })
      });
      const payload = await res.json();
      const [ newFen, llmMove ] = payload.data;

      // 3) apply the model’s move
      game.load(newFen);
      board.position(newFen);
      updateStatus(`LLM played ${llmMove}`);
    }
  });

  function updateStatus(msg) {
    document.getElementById("status").textContent = msg;
  }
</script>
