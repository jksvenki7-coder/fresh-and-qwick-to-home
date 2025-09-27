import React, { useState, useRef, useEffect } from "react";
import {
  APIProvider,
  Map,
  AdvancedMarker,
} from "@vis.gl/react-google-maps";

const GOOGLE_MAPS_API_KEY = "YOUR_API_KEY"; // Replace with your key

function ShopSelector({ onSelectShop }) {
  const [mapCenter, setMapCenter] = useState({ lat: 17.385, lng: 78.4867 }); // Example center
  const [selectedShop, setSelectedShop] = useState(null);

  // For demo, shops are hardcoded - these would come from your backend ideally
  const shops = [
    { id: 1, name: "Local Shop 1", lat: 17.385, lng: 78.4867 },
    { id: 2, name: "Local Shop 2", lat: 17.386, lng: 78.485 },
  ];

  const handleMarkerClick = (shop) => {
    setSelectedShop(shop);
    onSelectShop(shop);
  };

  return (
    <APIProvider apiKey={GOOGLE_MAPS_API_KEY}>
      <Map
        mapId="YOUR_MAP_ID" // Optional Map style id
        defaultCenter={mapCenter}
        defaultZoom={15}
        style={{ width: "100%", height: "300px" }}
      >
        {shops.map((shop) => (
          <AdvancedMarker
            key={shop.id}
            position={{ lat: shop.lat, lng: shop.lng }}
            onClick={() => handleMarkerClick(shop)}
            title={shop.name}
          />
        ))}
      </Map>
      {selectedShop && (
        <div>
          <strong>Selected Shop:</strong> {selectedShop.name}
        </div>
      )}
    </APIProvider>
  );
}

// Camera capture component using getUserMedia API
function CameraCapture({ onCapture }) {
  const videoRef = useRef(null);
  const canvasRef = useRef(null);
  const [streaming, setStreaming] = useState(false);

  useEffect(() => {
    async function startCamera() {
      if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        videoRef.current.srcObject = stream;
        videoRef.current.play();
        setStreaming(true);
      }
    }
    startCamera();
    return () => {
      if (videoRef.current && videoRef.current.srcObject) {
        videoRef.current.srcObject.getTracks().forEach((track) => track.stop());
      }
    };
  }, []);

  const capturePhoto = () => {
    const context = canvasRef.current.getContext("2d");
    context.drawImage(videoRef.current, 0, 0, 320, 240);
    const dataUrl = canvasRef.current.toDataURL("image/png");
    onCapture(dataUrl);
  };

  return (
    <div>
      {streaming ? (
        <>
          <video ref={videoRef} width="320" height="240" />
          <button onClick={capturePhoto}>Capture Photo</button>
          <canvas ref={canvasRef} width="320" height="240" style={{ display: "none" }} />
        </>
      ) : (
        <p>Loading camera...</p>
      )}
    </div>
  );
}

export default function ECommerceApp() {
  const [selectedCategory, setSelectedCategory] = useState(null);
  const [selectedShop, setSelectedShop] = useState(null);
  const [view, setView] = useState("home");
  const [capturedImage, setCapturedImage] = useState(null);

  const categories = [
    "Groceries",
    "Fruits & Vegetables",
    "Meat Items",
    "Milk Items & Snacks",
    "Flowers & Bouquets",
    "Doctors & Pharmacy",
    "Services",
    // Add more based on your content
  ];

  const handleSendWhatsApp = () => {
    if (!selectedShop) {
      alert("Please select a shop first");
      return;
    }
    // Format location and message for WhatsApp
    const message = encodeURIComponent(
      `Hello, I want to order from ${selectedShop.name} located here: https://www.google.com/maps/search/?api=1&query=${selectedShop.lat},${selectedShop.lng}`
    );
    const phone = "91XXXXXXXXXX"; // Replace with the shop's WhatsApp number or your number for demo
    const url = `https://wa.me/${phone}?text=${message}`;
    window.open(url, "_blank");
  };

  return (
    <div>
      <h1>E-Commerce Application</h1>

      {view === "home" && (
        <>
          <h2>Select Category</h2>
          <div>
            {categories.map((cat) => (
              <button key={cat} onClick={() => {
                setSelectedCategory(cat);
                setView("shopSelection");
              }}>
                {cat}
              </button>
            ))}
          </div>
          <button onClick={() => setView("camera")}>Service (Open Camera)</button>
        </>
      )}

      {view === "shopSelection" && (
        <>
          <h2>{selectedCategory} - Select Shop on Map</h2>
          <ShopSelector onSelectShop={setSelectedShop} />
          <br />
          <button onClick={handleSendWhatsApp} disabled={!selectedShop}>
            Send Location to WhatsApp
          </button>
          <br />
          <button onClick={() => {
            setView("home");
            setSelectedCategory(null);
            setSelectedShop(null);
          }}>Back to Categories</button>
        </>
      )}

      {view === "camera" && (
        <>
          <h2>Capture Service Image</h2>
          <CameraCapture onCapture={setCapturedImage} />
          {capturedImage && (
            <div>
              <h3>Captured Image:</h3>
              <img src={capturedImage} alt="Captured" width="320" height="240" />
            </div>
          )}
          <button onClick={() => {
            setCapturedImage(null);
            setView("home");
          }}>Back to Home</button>
        </>
      )}
    </div>
  );
}
