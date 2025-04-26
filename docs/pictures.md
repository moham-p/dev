<div class="gallery">
  <img id="gallery-image" src="" alt="Gallery Image">
  <div class="caption" id="gallery-caption"></div>
  <div class="buttons">
    <button onclick="prevImage()">Previous</button>
    <button onclick="nextImage()">Next</button>
  </div>
</div>

<script>
  const images = [
    { src: '/img/envista.jpg', caption: 'Envista' },
    { src: '/img/mapbox.jpg', caption: 'Mapbox' },
    { src: '/img/f-secure.jpg', caption: 'F-Secure' },
    { src: '/img/bankify.jpg', caption: 'Bankify' },
    { src: '/img/parsian.jpg', caption: 'Parsian Insurance' }
  ];

  let currentIndex = 0;

  function showImage(index) {
    const img = document.getElementById('gallery-image');
    const caption = document.getElementById('gallery-caption');
    img.src = images[index].src;
    img.alt = images[index].caption;
    caption.textContent = images[index].caption;
  }

  function prevImage() {
    currentIndex = (currentIndex - 1 + images.length) % images.length;
    showImage(currentIndex);
  }

  function nextImage() {
    currentIndex = (currentIndex + 1) % images.length;
    showImage(currentIndex);
  }

  showImage(currentIndex);
</script>

<style>
  .gallery {
    background: white;
    padding: 20px;
    border-radius: 8px;
    max-width: 600px;
    width: 90%;
    text-align: center;
    margin: auto;
  }
  .gallery img {
    width: 100%;
    height: auto;
    border-radius: 8px;
  }
  .caption {
    margin-top: 8px;
    font-size: 14px;
    color: #555;
  }
  .buttons {
    margin-top: 16px;
  }
  .buttons button {
    padding: 10px 20px;
    margin: 0 10px;
    font-size: 16px;
    cursor: pointer;
  }
</style>
