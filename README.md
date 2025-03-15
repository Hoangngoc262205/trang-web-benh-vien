# trang-web-benh-vien
trang web tìm kiếm bệnh viện xung quanh khu vực bắc từ liêm
import ReactDOM from "react-dom";
import NearestHospital from "./NearestHospital";

ReactDOM.render(<NearestHospital />, document.getElementById("root"));

/// NearestHospital.js
import { useEffect, useState } from "react";
import { getDistance, hospitals } from "./hospitalData";

export default function NearestHospital() {
  const [nearestHospital, setNearestHospital] = useState(null);

  useEffect(() => {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        (position) => {
          const userLat = position.coords.latitude;
          const userLng = position.coords.longitude;
          
          let closest = null;
          let minDistance = Number.MAX_VALUE;

          hospitals.forEach((hospital) => {
            const distance = getDistance(userLat, userLng, hospital.lat, hospital.lng);
            if (distance < minDistance) {
              minDistance = distance;
              closest = hospital;
            }
          });
          setNearestHospital(closest);
        },
        (error) => {
          console.error("Error fetching location", error);
        }
      );
    }
  }, []);

  return (
    <div>
      <h2>Tìm bệnh viện gần nhất</h2>
      {nearestHospital ? (
        <div>
          <p>Bệnh viện gần nhất: <strong>{nearestHospital.name}</strong></p>
          <a
            href={`https://www.google.com/maps/dir/?api=1&destination=${nearestHospital.lat},${nearestHospital.lng}`}
            target="_blank"
            rel="noopener noreferrer"
          >
            <button>Xem đường đi</button>
          </a>
          {nearestHospital.bookingUrl && (
            <a href={nearestHospital.bookingUrl} target="_blank" rel="noopener noreferrer">
              <button>Đặt lịch khám</button>
            </a>
          )}
        </div>
      ) : (
        <p>Đang xác định vị trí của bạn...</p>
      )}
    </div>
  );
}

/// hospitalData.js
export const hospitals = [
  {
    name: "Bệnh viện Phương Đông",
    lat: 21.0585,
    lng: 105.7634,
    bookingUrl: "https://phuongdonghospital.vn/dat-lich",
  },
  {
    name: "Bệnh viện 5 Sao Hà Nội",
    lat: 21.0285,
    lng: 105.8542,
    bookingUrl: "https://benhvien5sao.vn/dat-lich",
  },
];

export function getDistance(lat1, lon1, lat2, lon2) {
  const R = 6371;
  const dLat = ((lat2 - lat1) * Math.PI) / 180;
  const dLon = ((lon2 - lon1) * Math.PI) / 180;
  const a =
    Math.sin(dLat / 2) * Math.sin(dLat / 2) +
    Math.cos((lat1 * Math.PI) / 180) * Math.cos((lat2 * Math.PI) / 180) *
      Math.sin(dLon / 2) * Math.sin(dLon / 2);
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  return R * c;
}
