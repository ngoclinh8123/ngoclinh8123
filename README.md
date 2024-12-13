## Hi there 👋

<!--
**ngoclinh8123/ngoclinh8123** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->

⚡Did u know!!! The @ symbol, now vital for email, was originally just a random key with no real use. In fact, it used to be shorthand for “at the rate of” on typewriters! It only became famous when a programmer needed a symbol for email addresses and thought, "Hey, this @ thing isn't doing much... let's give it a purpose!"

<div id="header" align="center">
<!--   <img src="https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExcmQxMmVrOXk5bGQxNWJuYnhlc3d5ZzR5eWVlaXd2NmF1Y2V5MHN3NyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/11lxCeKo6cHkJy/giphy.gif" width="100"/> -->
  <img src="https://i.pinimg.com/originals/e5/21/25/e521256bba4e51f9212557b7beb09f80.gif" width="900"/>
</div>


<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Multi-Segment Creature</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      background-color: #f4f4f4;
    }
    canvas {
      display: block;
    }
  </style>
</head>
<body>
<canvas></canvas>
<script>
  const canvas = document.querySelector('canvas');
  const ctx = canvas.getContext('2d');
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;

  let mouse = { x: canvas.width / 2, y: canvas.height / 2 };

  // Creature class
  class Creature {
    constructor(x, y, segmentCount = 20, legCount = 4, legLength = 50) {
      this.segments = [];
      this.segmentSpacing = 20; // khoảng cách giữa các đốt
      this.legCount = legCount;
      this.legLength = legLength;
      this.speed = 0.1;
      this.bodyRadius = 20

      // Initialize segments
      for (let i = 0; i < segmentCount; i++) {
        const segmentX = x - i * this.segmentSpacing;
        const segmentY = y;
        this.segments.push({
          x: segmentX,
          y: segmentY,
          legs: this.createLegs(segmentX, segmentY, legCount),
        });
      }
    }
    
    // Create legs for a segment
    createLegs(segmentX, segmentY, legCount, segmentRadius = 20) {
      const legs = [];
      for (let i = 0; i < legCount; i++) {
        const angle = (i / legCount) * Math.PI * 2;
        
        // Adjust leg base to be positioned outside the segment, using segmentRadius
        const legBaseX = segmentX + Math.cos(angle) * segmentRadius;
        const legBaseY = segmentY + Math.sin(angle) * segmentRadius;
    
        // Calculate the target position of the leg based on legLength
        const targetX = segmentX + Math.cos(angle) * this.legLength;
        const targetY = segmentY + Math.sin(angle) * this.legLength;
    
        legs.push({
          baseAngle: angle,
          baseX: legBaseX,
          baseY: legBaseY,
          targetX: targetX,
          targetY: targetY,
          isPlanted: true,
        });
      }
      return legs;
    }

    // Update position and legs
    update(targetX, targetY) {
      const head = this.segments[0];

      // Update head position
      const deltaX = targetX - head.x;
      const deltaY = targetY - head.y;
      head.x += deltaX * this.speed;
      head.y += deltaY * this.speed;

      // Update other segments
      for (let i = 1; i < this.segments.length; i++) {
        const prevSegment = this.segments[i - 1];
        const segment = this.segments[i];
        const dx = prevSegment.x - segment.x;
        const dy = prevSegment.y - segment.y;
        const distance = Math.sqrt(dx ** 2 + dy ** 2);
        const angle = Math.atan2(dy, dx);

        // Keep segments spaced correctly
        if (distance > this.segmentSpacing) {
          segment.x = prevSegment.x - Math.cos(angle) * this.segmentSpacing;
          segment.y = prevSegment.y - Math.sin(angle) * this.segmentSpacing;
        }

        // Update legs for each segment
        this.updateLegs(segment, false);
      }

      // Update legs of the head segment (same as other segments now)
      this.updateLegs(head, true);
    }

    updateLegs(segment, isLeading) {
      segment.legs.forEach((leg) => {
        const currentLength = Math.sqrt(
          (leg.targetX - leg.baseX) ** 2 + (leg.targetY - leg.baseY) ** 2
        );
        const currentDistance = Math.sqrt((leg.targetX - segment.x) ** 2 + (leg.targetY - segment.y) ** 2)
        if (currentLength > this.legLength * 1.5 || currentDistance < this.bodyRadius) {
          leg.targetX = segment.x + Math.cos(leg.baseAngle) * this.legLength;
          leg.targetY = segment.y + Math.sin(leg.baseAngle) * this.legLength;
          leg.isPlanted = true;
        }
        
        leg.baseX = segment.x + Math.cos(leg.baseAngle) * 20;
        leg.baseY = segment.y + Math.sin(leg.baseAngle) * 20;
        
      });
    }
    

    draw() {
      ctx.fillStyle = '#800000';
      ctx.strokeStyle = '#B22222';
      ctx.lineWidth = 1;

      // Draw segments and legs
      this.segments.forEach((segment) => {
        // Draw legs
        segment.legs.forEach((leg) => {
          ctx.beginPath();
          ctx.moveTo(leg.baseX, leg.baseY); 
          ctx.lineTo(leg.targetX, leg.targetY);
          ctx.stroke();
        });

        // Draw segment body
        ctx.beginPath();
        ctx.arc(segment.x, segment.y, 20, 0, Math.PI * 2);
        ctx.fill();
      });
    }
  }

  const creature = new Creature(canvas.width / 2, canvas.height / 2);

  // Mouse move
  window.addEventListener('mousemove', (event) => {
    mouse.x = event.clientX;
    mouse.y = event.clientY;
  });

  // Animation loop
  function animate() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    creature.update(mouse.x, mouse.y);
    creature.draw();

    requestAnimationFrame(animate);
  }

  animate();
</script>
</body>
</html>
