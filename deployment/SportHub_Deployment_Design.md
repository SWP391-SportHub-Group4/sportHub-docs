# SportHub Deployment Design

**Phiên bản 1.0 · 17/11/2026**

Tài liệu mô tả nơi các thành phần SportHub chạy, cách chúng kết nối và cách phát hành, kiểm tra, khôi phục một bản triển khai. Thiết kế AWS được đối chiếu với bản `origin/main` lưu trong checkout tại commit `7fd6950b`. Trạng thái đang chạy trên AWS, cấu hình Vercel và chế độ truy cập PostgreSQL cần được xác nhận trên từng môi trường.

## 1. Kiến trúc triển khai

<div class="deployment-diagram">
<svg xmlns="http://www.w3.org/2000/svg" width="1200" height="860" viewBox="0 0 1200 860" role="img" aria-labelledby="title desc">
  <title id="title">SportHub deployment architecture with official AWS Architecture Icons</title>
  <desc id="desc">The browser calls Vercel and Caddy over HTTPS. Caddy, the ASP.NET API, PostgreSQL and FastAPI run in Docker Compose on one Amazon Lightsail server. The API calls Brevo, Cloudinary and VNPay. FastAPI calls Gemini.</desc>
  <metadata>Official AWS Architecture Icons package 07312026. Icons embedded without modifications. Source https://aws.amazon.com/architecture/icons/ . Architecture based on SportHub Deployment Design commit 7fd6950b.</metadata>
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto">
      <path d="M1 1 L9 5 L1 9" fill="none" stroke="#545B64" stroke-width="1.5" />
    </marker>
  </defs>
  <rect width="100%" height="100%" fill="#FFFFFF" />
  <g font-family="Inter, Arial, sans-serif">
    <text x="40" y="38" font-size="24" font-weight="700" text-anchor="start" fill="#16191F">SportHub · Kiến trúc triển khai</text>
    <rect x="290" y="220" width="650" height="590" fill="#FFFFFF" stroke="#242F3E" stroke-width="1.5" />
    <svg width="40" height="40" viewBox="0 0 40 40" version="1.1" x="290" y="220">
      <title>Icon-Architecture-Group/32/AWS-Cloud_32</title>
      <g id="aws-icon-0-Icon-Architecture-Group/32/AWS-Cloud_32" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <rect id="aws-icon-0-Rectangle" fill="#242F3E" x="0" y="0" width="40" height="40" />
        <path d="M28.993,27.9952689 L11.487,27.9762689 C9.121,27.9742689 7.154,26.1292689 7.012,23.7752689 C7.004,23.6522689 7,23.5262689 7,23.3972689 C7,20.2992689 9.091,19.2872689 10.337,18.9592689 C10.577,18.8962689 10.735,18.6662689 10.707,18.4192689 C10.676,18.1492689 10.659,17.8752689 10.659,17.5962689 C10.659,15.1322689 12.308,12.4822689 14.415,11.5632689 C15.359,11.1502689 16.232,10.9862689 17.023,10.9862689 C19.276,10.9862689 20.867,12.3152689 21.561,13.0332689 C22.329,13.8262689 22.927,14.8382689 23.34,16.0422689 C23.4,16.2192689 23.555,16.3472689 23.74,16.3742689 C23.918,16.4012689 24.109,16.3232689 24.219,16.1712689 C24.807,15.3502689 25.766,14.9822689 26.659,15.2392689 C27.782,15.5602689 28.516,16.7272689 28.62,18.3072689 C28.578,18.5762689 28.759,18.8292689 29.027,18.8772689 C30.222,19.0892689 33,19.9572689 33,23.4402689 C33,27.5892689 29.114,27.9822689 28.993,27.9952689 M29.594,17.9742689 C29.379,16.0672689 28.4,14.6962689 26.934,14.2782689 C25.899,13.9822689 24.811,14.2452689 23.989,14.9502689 C23.553,13.9362689 22.979,13.0602689 22.28,12.3382689 C20.023,10.0052689 16.933,9.37226889 14.014,10.6462689 C11.531,11.7302689 9.659,14.7182689 9.659,17.5962689 C9.659,17.7702689 9.665,17.9432689 9.676,18.1142689 C8.319,18.5732689 6,19.8752689 6,23.3972689 C6,23.5492689 6.004,23.6962689 6.014,23.8382689 C6.188,26.7162689 8.593,28.9732689 11.486,28.9762689 L29.034,28.9932689 C29.084,28.9892689 34,28.5192689 34,23.4402689 C34,19.5022689 31.003,18.3142689 29.594,17.9742689" id="aws-icon-0-Fill-1" fill="#FFFFFF" />
      </g>
    </svg>
    <text x="345" y="247" font-size="20" font-weight="700" text-anchor="start" fill="#16191F">AWS Cloud</text>
    <rect x="315" y="285" width="600" height="495" fill="#FFFFFF" stroke="#ED7100" stroke-width="1.5" />
    <svg width="56" height="56" viewBox="0 0 80 80" version="1.1" x="335" y="305">
      <title>Icon-Architecture/64/Arch_Amazon-Lightsail_64</title>
      <g id="aws-icon-1-Icon-Architecture/64/Arch_Amazon-Lightsail_64" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <g id="aws-icon-1-Icon-Architecture-BG/64/Compute" fill="#ED7100">
          <rect id="aws-icon-1-Rectangle" x="0" y="0" width="80" height="80" />
        </g>
        <path d="M32,41 L35,41 L35,39 L32,39 L32,41 Z M26,41 L29,41 L29,39 L26,39 L26,41 Z M20,41 L23,41 L23,39 L20,39 L20,41 Z M14,41 L17,41 L17,39 L14,39 L14,41 Z M59.627,47.412 L61.943,46.64 L59.627,45.868 L58.855,43.553 L58.084,45.868 L55.769,46.64 L58.084,47.412 L58.855,49.727 L59.627,47.412 Z M36.902,50.138 L36.13,52.454 L33.814,53.226 L36.13,53.997 L36.902,56.313 L37.674,53.997 L39.989,53.226 L37.674,52.454 L36.902,50.138 Z M36.902,25.258 L36.13,27.573 L33.814,28.345 L36.13,29.117 L36.902,31.433 L37.674,29.117 L39.989,28.345 L37.674,27.573 L36.902,25.258 Z M40.586,39.933 C44.257,42.338 46.837,47.031 47.9,53.252 C49.946,50.57 53.206,45.433 53.206,39.933 C53.206,34.459 50.325,29.647 47.907,26.573 C46.849,32.814 44.264,37.522 40.586,39.933 L40.586,39.933 Z M38.089,40.852 C37.722,40.694 37.483,40.333 37.483,39.933 C37.483,39.533 37.722,39.171 38.089,39.014 C42.459,37.14 45.426,31.705 46.228,24.104 C46.269,23.712 46.537,23.38 46.912,23.258 C47.286,23.135 47.699,23.246 47.964,23.539 C50.669,26.533 55.206,32.644 55.206,39.933 C55.206,47.676 49.663,54.432 47.964,56.318 C47.771,56.532 47.499,56.648 47.221,56.648 C47.117,56.648 47.014,56.632 46.912,56.6 C46.536,56.478 46.268,56.146 46.227,55.753 C45.424,48.157 42.458,42.725 38.089,40.852 L38.089,40.852 Z M67,40 C67,52.682 56.683,63 44,63 C32.553,63 22.756,54.46 21.212,43.135 L23.194,42.865 C24.604,53.204 33.548,61 44,61 C55.579,61 65,51.58 65,40 C65,28.42 55.579,19 44,19 C33.861,19 25.175,26.226 23.346,36.181 L21.379,35.819 C23.383,24.915 32.896,17 44,17 C56.683,17 67,27.318 67,40 L67,40 Z" id="aws-icon-1-Amazon-Lightsail-Icon_64_Squid" fill="#FFFFFF" />
      </g>
    </svg>
    <text x="410" y="326" font-size="21" font-weight="700" text-anchor="start" fill="#16191F">Amazon Lightsail</text>
    <text x="410" y="351" font-size="16" font-weight="400" text-anchor="start" fill="#545B64">Một máy chủ · Ubuntu 24.04</text>
    <rect x="335" y="390" width="560" height="365" fill="#FAFAFA" stroke="#687078" stroke-width="1.5" stroke-dasharray="7 5" />
    <text x="350" y="414" font-size="16" font-weight="400" text-anchor="start" fill="#545B64">Docker Compose</text>
    <rect x="40" y="90" width="200" height="120" fill="#FFFFFF" stroke="#AAB7B8" stroke-width="1.5" />
    <svg width="48" height="48" viewBox="0 0 48 48" version="1.1" x="116.0" y="102">
      <title>Icon-Resource/General-Resource/Res_Generic-Application_48_Light</title>
      <g id="aws-icon-2-Icon-Resource/General-Resource/Res_Generic-Application_48" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <path d="M37,35 L46,35 L46,37 L37,37 L37,46 L35,46 L35,37 L26,37 L26,35 L35,35 L35,26 L37,26 L37,35 Z M4,44 L20,44 L20,28 L4,28 L4,44 Z M21,26 L3,26 C2.447,26 2,26.448 2,27 L2,45 C2,45.552 2.447,46 3,46 L21,46 C21.553,46 22,45.552 22,45 L22,27 C22,26.448 21.553,26 21,26 L21,26 Z M28,20 L44,20 L44,4 L28,4 L28,20 Z M45,2 L27,2 C26.447,2 26,2.448 26,3 L26,21 C26,21.552 26.447,22 27,22 L45,22 C45.553,22 46,21.552 46,21 L46,3 C46,2.448 45.553,2 45,2 L45,2 Z M4,20 L20,20 L20,4 L4,4 L4,20 Z M21,2 L3,2 C2.447,2 2,2.448 2,3 L2,21 C2,21.552 2.447,22 3,22 L21,22 C21.553,22 22,21.552 22,21 L22,3 C22,2.448 21.553,2 21,2 L21,2 Z" id="aws-icon-2-Fill-1" fill="#232F3D" />
      </g>
    </svg>
    <text x="140.0" y="174" font-size="18" font-weight="700" text-anchor="middle" fill="#16191F">Vercel</text>
    <text x="140.0" y="197" font-size="15" font-weight="400" text-anchor="middle" fill="#545B64">Next.js 16 · Frontend</text>
    <rect x="40" y="440" width="200" height="120" fill="#FFFFFF" stroke="#AAB7B8" stroke-width="1.5" />
    <svg width="48" height="48" viewBox="0 0 48 48" version="1.1" x="116.0" y="452">
      <title>Icon-Resource/General-Resource/Res_Client_48_Light</title>
      <g id="aws-icon-3-Icon-Resource/General-Resource/Res_Client_48" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <path d="M42.299,38.773 L42.299,40.696 C42.299,41.134 41.945,41.49 41.506,41.49 L30.762,41.49 C30.323,41.49 29.968,41.134 29.968,40.696 L29.968,38.773 C29.968,38.335 30.323,37.98 30.762,37.98 L41.506,37.98 C41.945,37.98 42.299,38.335 42.299,38.773 L42.299,38.773 Z M43.797,42.399 C43.797,42.754 43.508,43.044 43.153,43.044 L4.645,43.044 C4.289,43.044 4,42.754 4,42.399 L4,37.011 C4,36.656 4.289,36.367 4.645,36.367 L43.153,36.367 C43.508,36.367 43.797,36.656 43.797,37.011 L43.797,42.399 Z M43.153,34.367 L4.645,34.367 C3.187,34.367 2,35.553 2,37.011 L2,42.399 C2,43.857 3.187,45.044 4.645,45.044 L43.153,45.044 C44.611,45.044 45.797,43.857 45.797,42.399 L45.797,37.011 C45.797,35.553 44.611,34.367 43.153,34.367 L43.153,34.367 Z M7.501,27.21 L40.264,27.21 L40.264,7.502 L7.501,7.502 L7.501,27.21 Z M40.842,5.502 L6.924,5.502 C6.14,5.502 5.501,6.139 5.501,6.923 L5.501,27.788 C5.501,28.572 6.14,29.21 6.924,29.21 L40.842,29.21 C41.626,29.21 42.264,28.572 42.264,27.788 L42.264,6.923 C42.264,6.139 41.626,5.502 40.842,5.502 L40.842,5.502 Z M43.797,30.067 C43.797,30.422 43.508,30.711 43.153,30.711 L4.645,30.711 C4.289,30.711 4,30.422 4,30.067 L4,4.644 C4,4.289 4.289,4 4.645,4 L43.153,4 C43.508,4 43.797,4.289 43.797,4.644 L43.797,30.067 Z M43.153,2 L4.645,2 C3.187,2 2,3.186 2,4.644 L2,30.067 C2,31.525 3.187,32.711 4.645,32.711 L43.153,32.711 C44.611,32.711 45.797,31.525 45.797,30.067 L45.797,4.644 C45.797,3.186 44.611,2 43.153,2 L43.153,2 Z" id="aws-icon-3-Fill-3" fill="#232F3D" />
      </g>
    </svg>
    <text x="140.0" y="524" font-size="18" font-weight="700" text-anchor="middle" fill="#16191F">Trình duyệt</text>
    <text x="140.0" y="547" font-size="15" font-weight="400" text-anchor="middle" fill="#545B64">Người dùng SportHub</text>
    <rect x="355" y="440" width="165" height="120" fill="#FFFFFF" stroke="#AAB7B8" stroke-width="1.5" />
    <svg width="48" height="48" viewBox="0 0 48 48" version="1.1" x="413.5" y="452">
      <title>Icon-Resource/General-Resource/Res_Generic-Application_48_Light</title>
      <g id="aws-icon-4-Icon-Resource/General-Resource/Res_Generic-Application_48" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <path d="M37,35 L46,35 L46,37 L37,37 L37,46 L35,46 L35,37 L26,37 L26,35 L35,35 L35,26 L37,26 L37,35 Z M4,44 L20,44 L20,28 L4,28 L4,44 Z M21,26 L3,26 C2.447,26 2,26.448 2,27 L2,45 C2,45.552 2.447,46 3,46 L21,46 C21.553,46 22,45.552 22,45 L22,27 C22,26.448 21.553,26 21,26 L21,26 Z M28,20 L44,20 L44,4 L28,4 L28,20 Z M45,2 L27,2 C26.447,2 26,2.448 26,3 L26,21 C26,21.552 26.447,22 27,22 L45,22 C45.553,22 46,21.552 46,21 L46,3 C46,2.448 45.553,2 45,2 L45,2 Z M4,20 L20,20 L20,4 L4,4 L4,20 Z M21,2 L3,2 C2.447,2 2,2.448 2,3 L2,21 C2,21.552 2.447,22 3,22 L21,22 C21.553,22 22,21.552 22,21 L22,3 C22,2.448 21.553,2 21,2 L21,2 Z" id="aws-icon-4-Fill-1" fill="#232F3D" />
      </g>
    </svg>
    <text x="437.5" y="524" font-size="18" font-weight="700" text-anchor="middle" fill="#16191F">Caddy</text>
    <text x="437.5" y="547" font-size="15" font-weight="400" text-anchor="middle" fill="#545B64">Container · TLS</text>
    <rect x="650" y="440" width="225" height="120" fill="#FFFFFF" stroke="#AAB7B8" stroke-width="1.5" />
    <svg width="48" height="48" viewBox="0 0 48 48" version="1.1" x="738.5" y="452">
      <title>Icon-Resource/General-Resource/Res_Generic-Application_48_Light</title>
      <g id="aws-icon-5-Icon-Resource/General-Resource/Res_Generic-Application_48" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <path d="M37,35 L46,35 L46,37 L37,37 L37,46 L35,46 L35,37 L26,37 L26,35 L35,35 L35,26 L37,26 L37,35 Z M4,44 L20,44 L20,28 L4,28 L4,44 Z M21,26 L3,26 C2.447,26 2,26.448 2,27 L2,45 C2,45.552 2.447,46 3,46 L21,46 C21.553,46 22,45.552 22,45 L22,27 C22,26.448 21.553,26 21,26 L21,26 Z M28,20 L44,20 L44,4 L28,4 L28,20 Z M45,2 L27,2 C26.447,2 26,2.448 26,3 L26,21 C26,21.552 26.447,22 27,22 L45,22 C45.553,22 46,21.552 46,21 L46,3 C46,2.448 45.553,2 45,2 L45,2 Z M4,20 L20,20 L20,4 L4,4 L4,20 Z M21,2 L3,2 C2.447,2 2,2.448 2,3 L2,21 C2,21.552 2.447,22 3,22 L21,22 C21.553,22 22,21.552 22,21 L22,3 C22,2.448 21.553,2 21,2 L21,2 Z" id="aws-icon-5-Fill-1" fill="#232F3D" />
      </g>
    </svg>
    <text x="762.5" y="524" font-size="18" font-weight="700" text-anchor="middle" fill="#16191F">SportHub API</text>
    <text x="762.5" y="547" font-size="15" font-weight="400" text-anchor="middle" fill="#545B64">ASP.NET Core 10</text>
    <rect x="355" y="630" width="200" height="120" fill="#FFFFFF" stroke="#AAB7B8" stroke-width="1.5" />
    <svg width="48" height="48" viewBox="0 0 48 48" version="1.1" x="431.0" y="642">
      <title>Icon-Resource/General-Resource/Res_Database_48_Light</title>
      <g id="aws-icon-6-Icon-Resource/General-Resource/Res_Database_48" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <path d="M31.755,43.307 C30.506,43.519 29.176,43.677 27.801,43.777 C27.244,43.817 26.678,43.847 26.105,43.869 C24.954,43.908 23.754,43.908 22.608,43.869 C22.033,43.847 21.467,43.817 20.91,43.777 C19.535,43.677 18.205,43.519 16.956,43.307 C12.049,42.475 9,40.73 9,38.754 L9,10.765 C11.997,12.661 17.74,13.841 24.356,13.841 C30.966,13.841 36.636,12.68 39.645,10.803 L39.645,38.754 C39.645,40.767 36.695,42.469 31.755,43.307 M24.356,4 C32.896,4 38.476,6.184 39.47,8.432 C37.802,10.098 32.46,11.841 24.356,11.841 C16.26,11.841 10.828,10.061 9.207,8.408 L9.171,8.443 C10.155,6.191 15.765,4 24.356,4 M24.356,2 C14.137,2 7,4.938 7,9.145 L7,38.754 C7,40.581 8.25,43.86 16.622,45.28 C17.933,45.501 19.327,45.667 20.766,45.771 C21.346,45.813 21.936,45.845 22.536,45.867 C23.134,45.888 23.741,45.899 24.356,45.899 C24.97,45.899 25.576,45.888 26.176,45.867 C26.776,45.845 27.365,45.813 27.945,45.771 C29.384,45.667 30.778,45.501 32.089,45.28 C40.403,43.869 41.645,40.586 41.645,38.754 L41.645,9.145 C41.645,4.938 34.535,2 24.356,2" id="aws-icon-6-Fill-9" fill="#232F3D" />
      </g>
    </svg>
    <text x="455.0" y="714" font-size="18" font-weight="700" text-anchor="middle" fill="#16191F">PostgreSQL 18</text>
    <text x="455.0" y="737" font-size="15" font-weight="400" text-anchor="middle" fill="#545B64">Container · Database</text>
    <rect x="675" y="630" width="200" height="120" fill="#FFFFFF" stroke="#AAB7B8" stroke-width="1.5" />
    <svg width="48" height="48" viewBox="0 0 48 48" version="1.1" x="751.0" y="642">
      <title>Icon-Resource/General-Resource/Res_Generic-Application_48_Light</title>
      <g id="aws-icon-7-Icon-Resource/General-Resource/Res_Generic-Application_48" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <path d="M37,35 L46,35 L46,37 L37,37 L37,46 L35,46 L35,37 L26,37 L26,35 L35,35 L35,26 L37,26 L37,35 Z M4,44 L20,44 L20,28 L4,28 L4,44 Z M21,26 L3,26 C2.447,26 2,26.448 2,27 L2,45 C2,45.552 2.447,46 3,46 L21,46 C21.553,46 22,45.552 22,45 L22,27 C22,26.448 21.553,26 21,26 L21,26 Z M28,20 L44,20 L44,4 L28,4 L28,20 Z M45,2 L27,2 C26.447,2 26,2.448 26,3 L26,21 C26,21.552 26.447,22 27,22 L45,22 C45.553,22 46,21.552 46,21 L46,3 C46,2.448 45.553,2 45,2 L45,2 Z M4,20 L20,20 L20,4 L4,4 L4,20 Z M21,2 L3,2 C2.447,2 2,2.448 2,3 L2,21 C2,21.552 2.447,22 3,22 L21,22 C21.553,22 22,21.552 22,21 L22,3 C22,2.448 21.553,2 21,2 L21,2 Z" id="aws-icon-7-Fill-1" fill="#232F3D" />
      </g>
    </svg>
    <text x="775.0" y="714" font-size="18" font-weight="700" text-anchor="middle" fill="#16191F">AI Service</text>
    <text x="775.0" y="737" font-size="15" font-weight="400" text-anchor="middle" fill="#545B64">FastAPI · Python 3.12</text>
    <rect x="980" y="440" width="195" height="120" fill="#FFFFFF" stroke="#AAB7B8" stroke-width="1.5" />
    <svg width="48" height="48" viewBox="0 0 48 48" version="1.1" x="1053.5" y="452">
      <title>Icon-Resource/General-Resource/Res_Generic-Application_48_Light</title>
      <g id="aws-icon-8-Icon-Resource/General-Resource/Res_Generic-Application_48" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <path d="M37,35 L46,35 L46,37 L37,37 L37,46 L35,46 L35,37 L26,37 L26,35 L35,35 L35,26 L37,26 L37,35 Z M4,44 L20,44 L20,28 L4,28 L4,44 Z M21,26 L3,26 C2.447,26 2,26.448 2,27 L2,45 C2,45.552 2.447,46 3,46 L21,46 C21.553,46 22,45.552 22,45 L22,27 C22,26.448 21.553,26 21,26 L21,26 Z M28,20 L44,20 L44,4 L28,4 L28,20 Z M45,2 L27,2 C26.447,2 26,2.448 26,3 L26,21 C26,21.552 26.447,22 27,22 L45,22 C45.553,22 46,21.552 46,21 L46,3 C46,2.448 45.553,2 45,2 L45,2 Z M4,20 L20,20 L20,4 L4,4 L4,20 Z M21,2 L3,2 C2.447,2 2,2.448 2,3 L2,21 C2,21.552 2.447,22 3,22 L21,22 C21.553,22 22,21.552 22,21 L22,3 C22,2.448 21.553,2 21,2 L21,2 Z" id="aws-icon-8-Fill-1" fill="#232F3D" />
      </g>
    </svg>
    <text x="1077.5" y="524" font-size="18" font-weight="700" text-anchor="middle" fill="#16191F">Brevo · Cloudinary</text>
    <text x="1077.5" y="547" font-size="15" font-weight="400" text-anchor="middle" fill="#545B64">VNPay</text>
    <rect x="980" y="630" width="195" height="120" fill="#FFFFFF" stroke="#AAB7B8" stroke-width="1.5" />
    <svg width="48" height="48" viewBox="0 0 48 48" version="1.1" x="1053.5" y="642">
      <title>Icon-Resource/General-Resource/Res_Generic-Application_48_Light</title>
      <g id="aws-icon-9-Icon-Resource/General-Resource/Res_Generic-Application_48" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <path d="M37,35 L46,35 L46,37 L37,37 L37,46 L35,46 L35,37 L26,37 L26,35 L35,35 L35,26 L37,26 L37,35 Z M4,44 L20,44 L20,28 L4,28 L4,44 Z M21,26 L3,26 C2.447,26 2,26.448 2,27 L2,45 C2,45.552 2.447,46 3,46 L21,46 C21.553,46 22,45.552 22,45 L22,27 C22,26.448 21.553,26 21,26 L21,26 Z M28,20 L44,20 L44,4 L28,4 L28,20 Z M45,2 L27,2 C26.447,2 26,2.448 26,3 L26,21 C26,21.552 26.447,22 27,22 L45,22 C45.553,22 46,21.552 46,21 L46,3 C46,2.448 45.553,2 45,2 L45,2 Z M4,20 L20,20 L20,4 L4,4 L4,20 Z M21,2 L3,2 C2.447,2 2,2.448 2,3 L2,21 C2,21.552 2.447,22 3,22 L21,22 C21.553,22 22,21.552 22,21 L22,3 C22,2.448 21.553,2 21,2 L21,2 Z" id="aws-icon-9-Fill-1" fill="#232F3D" />
      </g>
    </svg>
    <text x="1077.5" y="714" font-size="18" font-weight="700" text-anchor="middle" fill="#16191F">Gemini API</text>
    <text x="1077.5" y="737" font-size="15" font-weight="400" text-anchor="middle" fill="#545B64">Dịch vụ AI bên ngoài</text>
    <polyline points="140,440 140,210" fill="none" stroke="#545B64" stroke-width="1.6" marker-end="url(#arrow)" />
    <rect x="150.2" y="315" width="85.60000000000001" height="19" fill="#FFFFFF" />
    <text x="193" y="330" font-size="15" font-weight="400" text-anchor="middle" fill="#545B64">HTTPS 443</text>
    <polyline points="240,500 355,500" fill="none" stroke="#545B64" stroke-width="1.6" marker-end="url(#arrow)" />
    <rect x="254.2" y="470" width="85.60000000000001" height="19" fill="#FFFFFF" />
    <text x="297" y="485" font-size="15" font-weight="400" text-anchor="middle" fill="#545B64">HTTPS 443</text>
    <polyline points="520,500 650,500" fill="none" stroke="#545B64" stroke-width="1.6" marker-end="url(#arrow)" />
    <rect x="542.2" y="470" width="85.60000000000001" height="19" fill="#FFFFFF" />
    <text x="585" y="485" font-size="15" font-weight="400" text-anchor="middle" fill="#545B64">HTTP 8080</text>
    <polyline points="875,500 980,500" fill="none" stroke="#545B64" stroke-width="1.6" marker-end="url(#arrow)" />
    <rect x="884.2" y="470" width="85.60000000000001" height="19" fill="#FFFFFF" />
    <text x="927" y="485" font-size="15" font-weight="400" text-anchor="middle" fill="#545B64">HTTPS 443</text>
    <polyline points="705,560 705,595 455,595 455,630" fill="none" stroke="#545B64" stroke-width="1.6" marker-end="url(#arrow)" />
    <rect x="521.4" y="568" width="77.2" height="19" fill="#FFFFFF" />
    <text x="560" y="583" font-size="15" font-weight="400" text-anchor="middle" fill="#545B64">TCP 5432</text>
    <polyline points="800,560 800,630" fill="none" stroke="#545B64" stroke-width="1.6" marker-end="url(#arrow)" />
    <rect x="804.2" y="587" width="85.60000000000001" height="19" fill="#FFFFFF" />
    <text x="847" y="602" font-size="15" font-weight="400" text-anchor="middle" fill="#545B64">HTTP 8001</text>
    <polyline points="875,690 980,690" fill="none" stroke="#545B64" stroke-width="1.6" marker-end="url(#arrow)" />
    <rect x="884.2" y="660" width="85.60000000000001" height="19" fill="#FFFFFF" />
    <text x="927" y="675" font-size="15" font-weight="400" text-anchor="middle" fill="#545B64">HTTPS 443</text>
  </g>
</svg>
</div>

Các thành phần trong khung Docker Compose chạy trên cùng một máy Amazon Lightsail. Hướng mũi tên thể hiện bên khởi tạo kết nối. Sơ đồ được nhúng bằng mã SVG nội tuyến, với bố cục, màu sắc và AWS Architecture Icons của bản thiết kế gốc.

[Nguồn biểu tượng AWS Architecture Icons](https://aws.amazon.com/architecture/icons/).

| Thành phần | Nơi chạy | Vai trò khi triển khai |
| --- | --- | --- |
| Frontend | Vercel, thư mục `frontend/`, Next.js 16 và React 19 | Phục vụ giao diện. Mã chạy trên trình duyệt gọi trực tiếp API AWS qua HTTPS. |
| Caddy | Container `caddy` trên Lightsail | Nhận truy cập cổng `80/443`, xử lý HTTPS và chuyển yêu cầu đến `backend:8080`. DNS của miền API trỏ về máy chủ Lightsail. |
| Backend | Container `backend` trên Lightsail | API ASP.NET Core 10 dạng modular monolith. Các tác vụ nền về hết hạn, thông báo và đối soát chạy cùng tiến trình API. |
| Cơ sở dữ liệu | Container `postgres` trên Lightsail | PostgreSQL 18 (`postgres:18-bookworm`). Backend kết nối qua `postgres:5432`. |
| AI Service | Container `ai-service` trên Lightsail | FastAPI trên Python 3.12 và Uvicorn. Backend gọi `http://ai-service:8001`, dịch vụ này gọi Gemini. |
| Khởi tạo lưu trữ | Container `storage-init` trên Lightsail | Chuẩn bị thư mục và quyền ghi cho volume chứa khóa Data Protection và báo cáo trước khi backend khởi động. |

Runbook ghi nhận máy Lightsail có 2 vCPU, 2 GB RAM và 60 GB đĩa. Backend và AI image được build trên GitHub Actions, giúp giảm tải cho máy chủ khi phát hành. Một máy chủ vận hành cả proxy, API, AI và DB. Khi máy chủ gặp sự cố, các chức năng phụ thuộc API có thể gián đoạn cùng lúc. Mỗi lần cập nhật AWS có khoảng gián đoạn khi sao lưu, chạy migration và khởi động lại.

## 2. Môi trường chạy

| Môi trường | Thành phần và cấu hình | Dữ liệu |
| --- | --- | --- |
| Phát triển local | Docker Compose cho PostgreSQL 16 và AI. API có thể chạy bằng .NET SDK 10 ở `Development` và frontend bằng Node.js 24 với `npm run dev`. Địa chỉ mặc định `localhost:3000` (web), `localhost:5000` (API), `localhost:5435` (DB), `127.0.0.1:8001` (AI). | DB local riêng. Chạy toàn bộ stack bằng Compose cũng được hỗ trợ. |
| Kiểm thử CI | GitHub Actions chạy build/test backend, lint/typecheck/build frontend, Playwright và kiểm tra script triển khai. E2E dùng PostgreSQL 16 riêng và API ở `Development`. | Database `sporthub_e2e`, với dữ liệu test được tạo trên runner. |
| Production | Vercel phục vụ frontend. Ubuntu 24.04 trên Lightsail chạy Compose project `sporthub-aws` với API ở `Production`, AI và PostgreSQL 18. | Dữ liệu vận hành và tệp runtime lưu trên Docker volumes của máy AWS. |

### Chạy local

1. Tạo `.env` ở gốc repo từ `.env.example` nếu tệp chưa tồn tại. Điền cấu hình PostgreSQL và khóa JWT local. Với API chạy trực tiếp, `ConnectionStrings__Default` trỏ về `localhost:5435`. Để kiểm tra các luồng AI cần Gemini, cấu hình `ai-service/.env` với `GEMINI_API_KEY`.
2. Khởi động `docker compose up -d postgres ai-service`. Nếu cần đổi cổng AI thì đồng bộ cấu hình API với cổng mới.
3. Từ gốc repo, chạy API trong PowerShell:

   ```powershell
   dotnet run --project backend/SportHub.API/SportHub.API.csproj -c Release -- --environment=Development --urls=http://localhost:5000 --DataProtection:KeysPath="$PWD/.tmp/local-api-keys"
   ```

4. Trong thư mục `frontend`, chạy `npm ci` rồi `npm run dev`. Nếu đổi cổng API, cập nhật `NEXT_PUBLIC_API_BASE_URL` trong `frontend/.env.local` rồi khởi động lại frontend.
5. Mở `http://localhost:3000` và kiểm tra `http://localhost:5000/api/health`. Có thể chạy toàn bộ hệ thống bằng `docker compose up -d --build` sau khi chuẩn bị các tệp cấu hình local.

### Kết nối cơ sở dữ liệu từ xa

Ở cấu hình AWS mặc định (`POSTGRES_ACCESS=private` trên `origin/main`), chỉ Caddy publish cổng ứng dụng ra host. SSH quản trị dùng cổng `22` theo phạm vi nguồn được cấp quyền. Các cổng nội bộ `5432`, `8001` và `8080` không được publish. Khi cần dùng DB từ ngoài host, repo mô tả hai phương án:

| Phương án | Đường kết nối | Điều kiện |
| --- | --- | --- |
| SSH tunnel | Máy dev `127.0.0.1:15432` → SSH → PostgreSQL `5432` trong mạng Docker | Áp dụng với profile `private`. Mỗi thành viên sử dụng tài khoản DB riêng. |
| Kết nối trực tiếp có TLS | Máy dev → AWS TCP `15432` → PostgreSQL `5432` | Operator phải bật profile `public`, chuẩn bị chứng chỉ và thay đổi firewall. PostgreSQL dùng TLS và SCRAM. Client dùng `SSL Mode=VerifyFull` với CA được cấp. Việc merge mã không tự bật cổng này. |

Khi trỏ API local vào DB website, chạy API trực tiếp với `SharedDatabase__Enabled=true` theo hướng dẫn repo. Chế độ này tắt migration, seed và tác vụ nền ở local. Request ghi dữ liệu vẫn tác động đến DB thật, nên kiểm thử tự động dùng database riêng của local hoặc CI.

## 3. Cấu hình production

| Phạm vi | Thiết lập |
| --- | --- |
| Vercel | Root Directory `frontend`, Production Branch `main`, lệnh `npm ci` và `npm run build`. Đặt `NEXT_PUBLIC_API_BASE_URL=https://<api-domain>` trước khi build. Nếu dùng Google Sign-In, đặt `NEXT_PUBLIC_GOOGLE_CLIENT_ID` khớp `GOOGLE_CLIENT_ID` ở backend và cho phép origin Vercel ở cấu hình Google. |
| Caddy và CORS | `API_DOMAIN` chỉ chứa hostname. `ACME_EMAIL` dùng để cấu hình TLS. `FRONTEND_ORIGIN` là origin HTTPS chính xác của Vercel, không có dấu `/` ở cuối. Compose dùng biến này cho CORS, `Frontend__BaseUrl` và URL trả về sau thanh toán. |
| PostgreSQL và xác thực | `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD` và `JWT_SECRET`. Backend kết nối bằng tên dịch vụ nội bộ `postgres`. Giữ JWT issuer/audience/secret phù hợp với cấu hình của môi trường khi cập nhật. |
| Email, lưu trữ và AI | Backend dùng `BREVO_API_KEY`, `BREVO_FROM_ADDRESS` và `CLOUDINARY_URL`. AI service dùng `GEMINI_API_KEY` và `GEMINI_MODEL`. |
| VNPay | Thiết lập `VNPAY_TMN_CODE`, `VNPAY_HASH_SECRET`, `VNPAY_PAYMENT_URL` và `VNPAY_QUERY_URL` theo môi trường merchant. Compose mặc định URL sandbox nếu không ghi đè. Trình duyệt quay về frontend `/payments/return`, còn IPN trỏ đến API `/api/payments/vnpay/ipn` khi merchant đã cấu hình callback. |

Trên AWS, tạo `deploy/aws/.env` từ mẫu và cấp quyền `0600`. Giữ thông tin xác thực, khóa và bản sao cấu hình production ngoài Git, GitHub Releases và log công khai. Các biến `NEXT_PUBLIC_*` được đưa vào bản build frontend, vì vậy cần build lại khi đổi API URL hoặc Google client ID.

## 4. Phát hành và cập nhật

### Chuẩn bị ban đầu

1. Chuẩn bị host Ubuntu, Docker Engine và Compose. Trỏ DNS miền API về host. Cấu hình firewall theo các cổng cần dùng.
2. Cấu hình `deploy/aws/.env` và kiểm tra Compose bằng `bash deploy/aws/compose.sh config --quiet`. Dùng runbook AWS để chuẩn bị database và dữ liệu ban đầu. Khi chuyển hệ thống hiện hữu từ Railway, tạm dừng ghi ở nguồn, phục hồi dump PostgreSQL và tệp runtime vào đích trống, rồi đối chiếu dữ liệu trước khi chạy migration.
3. Với luồng chuyển từ Railway, lệnh được runbook chỉ định sau khi xác minh restore là `bash deploy/aws/deploy.sh --confirm-restored`. Kiểm tra API và tích hợp trước khi đổi URL API trên Vercel và phát hành frontend.
4. Với hệ thống AWS đã được cài đặt, chạy `deploy/aws/setup-cd.sh` một lần theo `CD.md` để kích hoạt cập nhật tự động.

### Phát hành thường xuyên từ `main`

1. GitHub Actions chạy kiểm tra trên PR và các nhánh cấu hình trong workflow. Job phát hành AWS chỉ chạy cho `main` ngoài sự kiện PR, sau khi backend, frontend, E2E và deployment checks đều đạt.
2. Actions build hai image backend và AI theo full commit SHA, đóng gói source, mã phiên bản và checksum, rồi công bố GitHub Release `aws-<FULL_SHA>`.
3. `sporthub-cd.timer` trên AWS kiểm tra release khoảng 5 phút một lần sau khi kích hoạt. CD kiểm tra commit `main`, trạng thái CI, nguồn phát hành và checksum trước khi cập nhật.
4. CD dừng Caddy và backend để sao lưu cùng cấu hình, chạy EF Core migration qua `/app/migrate`, rồi khởi động lại backend, AI và Caddy. CD xác nhận release sau khi HTTPS, API đọc DB và CORS đều đạt.
5. Vercel phát hành frontend độc lập. Khi thay đổi API, ưu tiên giữ tương thích với cả frontend đang chạy và bản mới trong thời gian hai môi trường cập nhật lệch nhau.

## 5. Xác nhận triển khai

| Kiểm tra | Kết quả cần thấy |
| --- | --- |
| Trạng thái release | `journalctl -u sporthub-cd.service -n 50 --no-pager` ghi `DEPLOYMENT_SUCCEEDED <FULL_SHA>` hoặc `UP_TO_DATE`. `bash /opt/sporthub/app/deploy/aws/compose.sh ps` hiển thị các dịch vụ chính đang chạy. |
| Proxy, API và DB | `GET https://<api-domain>/api/health` trả HTTP 200 với `status: ok`. `GET /api/classes` qua cùng miền trả HTTP 200 và xác nhận API đọc được DB. |
| CORS | Preflight `OPTIONS /api/auth/login` với origin của frontend nhận `Access-Control-Allow-Origin` đúng origin đã cấu hình. |
| Nghiệp vụ | Frontend mở được, đăng nhập bằng tài khoản kiểm thử, kiểm tra đúng vai trò và xác nhận ít nhất một luồng đặt sân hoặc đăng ký lớp. |
| Tích hợp | Thử một yêu cầu AI thực, gửi một email kiểm thử, kiểm tra ảnh hoặc tệp trên Cloudinary, và xác nhận VNPay trong sandbox gồm trang trả về, IPN nếu được bật và trạng thái hóa đơn sau đối soát. |

Log container dùng `json-file` với giới hạn `10 MB` mỗi tệp và tối đa ba tệp. Log triển khai chi tiết ở `/opt/sporthub/cd-logs/` và chỉ dành cho quản trị máy chủ. Người dùng vai trò Center Manager có thể xem tình trạng job nền, hàng đợi thông báo và đối soát qua `GET /api/manager/operations/health`.

## 6. Lưu trữ, sao lưu và khôi phục

| Dữ liệu | Vị trí hoặc định dạng |
| --- | --- |
| Dữ liệu nghiệp vụ | Volume `postgres_data`, gắn tại `/var/lib/postgresql` theo bố trí dữ liệu của PostgreSQL 18. |
| Báo cáo và dữ liệu mã hóa | Volume `backend_data`, gắn tại `/var/lib/sporthub`, chứa `reports` và `dataprotection-keys`. Các khóa cần đi cùng DB để worker đọc lại nội dung outbox đã mã hóa. |
| Trạng thái Caddy | Volumes `caddy_data` và `caddy_config`. |
| Bản sao trước mỗi lần CD | `/opt/sporthub-backups/cd-…/` chứa `production.env` riêng tư và thư mục con có `database.dump`, `backend-data.tar` và `SHA256SUMS`. Lưu thêm bản mã hóa ngoài máy và thử restore trên đích trống riêng. |

- **Lỗi trước migration.** Nếu lỗi ở bước chuẩn bị, hệ thống tiếp tục chạy bản hiện hành. Nếu lỗi khi backup sau khi đã dừng dịch vụ, CD thử khởi động lại bản cũ.
- **Lỗi migration, khởi động hoặc health check.** CD dừng API/Caddy, giữ DB và backup, đồng thời chặn lần cập nhật tiếp theo bằng `cd-blocked.json`. Operator kiểm tra schema và log rồi chọn bản ứng dụng tương thích hoặc phục hồi sang đích trống từ bộ backup tương ứng. Sau khi xác minh dữ liệu và xử lý nguyên nhân, operator mới gỡ chặn CD.
- **Quay về Railway sau khi AWS đã nhận dữ liệu.** Tạm dừng ghi, sao lưu AWS và lập kế hoạch chuyển ngược hoặc đối soát dữ liệu phát sinh trước khi đổi điểm truy cập. Database Railway không đồng bộ với AWS.

## Nguồn cấu hình

- [Cấu hình Compose local](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/7fd6950b07d6e43f5267c005c759bd460763b429/docker-compose.yml) · [Cấu hình Compose AWS](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/7fd6950b07d6e43f5267c005c759bd460763b429/deploy/aws/compose.yml) · [Cấu hình Vercel trong repo](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/7fd6950b07d6e43f5267c005c759bd460763b429/frontend/vercel.json)
- [Workflow CI/CD](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/7fd6950b07d6e43f5267c005c759bd460763b429/.github/workflows/ci.yml) · [Hướng dẫn CD trên AWS](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/7fd6950b07d6e43f5267c005c759bd460763b429/deploy/aws/CD.md) · [Runbook AWS](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/7fd6950b07d6e43f5267c005c759bd460763b429/deploy/aws/RUNBOOK.txt)
- [Truy cập DB bằng SSH tunnel](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/7fd6950b07d6e43f5267c005c759bd460763b429/deploy/aws/LOCAL-DATABASE.md) · [Endpoint PostgreSQL có TLS](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/7fd6950b07d6e43f5267c005c759bd460763b429/deploy/aws/PUBLIC-DATABASE.md)

