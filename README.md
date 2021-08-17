Triển khai call video WebRTC và NodeJS

Giới thiệu WebRTC

WebRTC (Web Real Time Communication) là một dự án mã nguồn mở cho phép giao tiếp ngang hàng giữa các trình duyệt. Nói cách khác, WebRTC cho phép bạn trao đổi bất kỳ loại phương tiện nào thông qua web (chẳng hạn như video, âm thanh và dữ liệu) mà không cần bất kỳ plugin hoặc khuôn khổ bắt buộc nào.

Giao tiếp trực tiếp giữa các trình duyệt giúp cải thiện hiệu suất và giảm thời gian chờ vì máy khách không cần tiếp tục gửi và nhận tin nhắn thông qua máy chủ. 
Ví dụ: chúng ta có thể sử dụng WebSockets để kết nối hai máy khách nhưng một máy chủ sẽ phải định tuyến các thông điệp của chúng như trong sơ đồ tiếp theo:

Ngược lại, WebRTC chỉ cần một máy chủ để thiết lập và kiểm soát các kết nối của máy khách. Quá trình này được gọi là tín hiệu. Khi các trình duyệt đã thu thập thông tin cần thiết của các trình duyệt ngang hàng, chúng có thể giao tiếp với nhau:

 WebRTC không chỉ định giao thức nhắn tin báo hiệu nên việc triển khai đã chọn phải dựa trên các yêu cầu của ứng dụng, một số cách tiếp cận phổ biến nhất là: WebSockets, SIP, XHR, XMPP, v.v.v 

Quá trình báo hiệu hoạt động như sau:

Một khách hàng bắt đầu cuộc gọi.

Người gọi tạo một offer bằng Giao thức mô tả phiên (SDP) và gửi offer đó cho người ngang hàng khác (callee).
Người phụ trách phản hồi offer bằng một tin nhắn trả lời có chứa cả mô tả SDP.
Sau khi cả hai peers đã đặt mô tả phiên cục bộ và từ xa, bao gồm thông tin như codec và siêu dữ liệu của trình duyệt, họ biết các khả năng phương tiện được sử dụng cho cuộc gọi. Tuy nhiên, họ sẽ chưa thể kết nối và trao đổi dữ liệu phương tiện của mình vì các SDP không biết về những thứ như NAT bên ngoài (Trình biên dịch địa chỉ mạng), địa chỉ IP và cách xử lý bất kỳ hạn chế nào về cổng. Điều này đạt được nhờ Tổ chức Kết nối Tương tác (ICE).

 Vậy Thiết lập Kết nối Tương tác hoạt động như thế nào? ICE là một phương thức truyền thông mạng ngang hàng để gửi và nhận thông tin phương tiện qua Internet. Cách định tuyến được thực hiện (NAT, tường lửa…) nằm ngoài phạm vi của bài viết này nhưng đó là điều mà WebRTC cần giải quyết. ICE tập hợp các kết nối mạng có sẵn, được gọi là ứng viên ICE và sử dụng các giao thức STUN (Tiện ích truyền phiên cho NAT) và TURN (Chuyển tiếp sử dụng chuyển tiếp xung quanh NAT) cho NAT và truyền tải tường lửa.

Những bài tiếp theo mình sẽ hướng dẫn chi tiết về Stun server:

Đây là cách ICE xử lý kết nối:

Đầu tiên, nó cố gắng kết nối các peers trực tiếp thông qua UDP.
Nếu UDP không thành công, nó sẽ thử TCP.
Nếu cả kết nối trực tiếp UDP và TCP đều không thành công, điều này thường xảy ra trong các tình huống thực do NAT và tường lửa, ICE trước tiên sẽ sử dụng máy chủ STUN với UDP để kết nối các peers. Máy chủ STUN là máy chủ thực hiện giao thức STUN và được sử dụng để tìm địa chỉ công cộng và cổng của một máy ngang hàng đằng sau NAT bất đối xứng.
Nếu máy chủ STUN bị lỗi, ICE sẽ sử dụng máy chủ TURN, là máy chủ STUN có thêm một số chức năng chuyển tiếp có thể truyền qua NAT đối xứng.
Như bạn có thể thấy, ICE cố gắng sử dụng máy chủ STUN trước nhưng máy chủ chuyển tiếp TURN sẽ cần thiết cho các mạng công ty rất hạn chế. Máy chủ chuyển tiếp TURN đắt tiền, bạn sẽ cần phải trả tiền cho máy chủ của riêng mình hoặc sử dụng nhà cung cấp dịch vụ, tuy nhiên, hầu hết thời gian ICE sẽ có thể kết nối các máy chủ ngang hàng với STUN. Lược đồ sau cho thấy một giao tiếp STUN / TURN:



Tóm lại, quá trình báo hiệu được sử dụng để trao đổi thông tin phương tiện với các tệp SDP và ICE được sử dụng để trao đổi kết nối mạng của các peers. Sau khi giao tiếp đã được thiết lập, các peers cuối cùng có thể trao đổi dữ liệu trực tiếp thông qua trình duyệt của nó.

Về bảo mật, WebRTC bao gồm một số tính năng bắt buộc để đối phó với các rủi ro khác nhau:

Các luồng phương tiện được mã hóa bằng Giao thức truyền tải thời gian thực bảo mật (SRTP) và các luồng dữ liệu được mã hóa bằng Bảo mật lớp truyền tải Datagram (DTLS).
User phải cấp quyền truy cập vào máy ảnh và micrô. Để khách hàng biết, các trình duyệt sẽ hiển thị các biểu tượng nếu máy ảnh hoặc micrô của thiết bị đang hoạt động.
Tất cả các thành phần WebRTC đều chạy trong browser sandbox  của trình duyệt và sử dụng mã hóa, chúng không cần bất kỳ loại cài đặt nào, chúng sẽ chỉ hoạt động miễn là trình duyệt hỗ trợ chúng.

WebRTC của API


WebRTC dựa trên ba API JavaScript chính:

MediaStream (hay còn gọi là getUserMedia): giao diện này đại diện cho luồng phương tiện của thiết bị có thể bao gồm các bản âm thanh và video. Phương thức MediaDevies.getUserMedia () truy xuất MediaStream (ví dụ: nó có thể được sử dụng để truy cập máy ảnh của điện thoại).


RTCPeerConnection: nó cho phép giao tiếp giữa các peers. Các luồng do MediaDevices.getUserMedia () truy cập được thêm vào thành phần này, thành phần này cũng xử lý các tin nhắn trả lời và đề nghị SDP được trao đổi giữa các peers và ứng viên ICE.


RTCDataChannel: nó cho phép truyền dữ liệu tùy ý theo thời gian thực. Nó thường được so sánh với WebSockets mặc dù nó kết nối các trình duyệt để trao đổi dữ liệu trực tiếp. Như đã giải thích trước đây, giao tiếp trực tiếp giữa các trình duyệt cải thiện hiệu suất để API này có thể được sử dụng cho một số ứng dụng thú vị như chơi game hoặc chia sẻ tệp được mã hóa.

Ở bài ví dụ hôm này thì mình sẽ triển khai WebRTC với Node chỉ dùng tới 2 API đó là getUserMedia và RTCPeerConnection.

Triển khai call video webRTC với Node

Trong phần này, chúng ta sẽ triển khai ứng dụng trò chuyện video với nhiều phòng mà người dùng có thể chọn. Nếu hai khách hàng kết nối với cùng một phòng, họ sẽ bắt đầu trò chuyện và truyền video cho nhau. Vì đây là một ví dụ đơn giản với mục đích chỉ là cho thấy cách hoạt động của WebRTC, vẫn còn chỗ cho nhiều cải tiến và tính năng.

Tạo cấu trúc ứng dụng
Trước tiên, chúng tôi sẽ tạo thư mục của dự án và bắt đầu:

mkdir webrtc-node-app && cd webrtc-node-app
npm init

Cấu trúc của ứng dụng của chúng ta sẽ là:

server.js
public/
|_index.html
|_client.js

Triển khai máy chủ
Ở đây mình sẽ sử dụng Express làm Frameworks Node và SocketIO làm thư viện JavaScript để giao tiếp realtime giữa máy client và máy server. Trong khi SocketIO là một thư viện để làm việc với WebSockets, nó hỗ trợ một số tính năng bổ sung như phát sóng.

Bắt đầu cài đặt các thư viện:

npm cài đặt express socket.io

Tệp server.js sẽ chạy ứng dụng trên cổng 1483 và xử lý các thông báo WebSockets sẽ được sử dụng để báo hiệu (như đã thảo luận trước đó, đó là cách các peers sẽ trao đổi thông tin phương tiện của họ).

Trong tệp server.js, hãy tạo một classic Express Server và add middleware để có quyền truy cập vào thư mục chung:


const express = require('express')
const app = express()
const server = require('http').createServer(app)
 
app.use('/', express.static('public'))
 
// START THE SERVER ==========================================================
const port = process.env.PORT || 1483
server.listen(port, () => {
  console.log(`Express server listening on port ${port}`)
})
Sau đó import thư viện SocketIO và xử lý các thông báo sẽ được gửi bởi các máy client:

const express = require('express')
const app = express()
const server = require('http').createServer(app)
const io = require('socket.io')(server)
 
app.use('/', express.static('public'))
 
io.on('connection', (socket) => {
  socket.on('join', (roomId) => {
    const selectedRoom = io.sockets.adapter.rooms[roomId]
    const numberOfClients = selectedRoom ? selectedRoom.length : 0
 
    // These events are emitted only to the sender socket.
    if (numberOfClients == 0) {
      console.log(`Creating room ${roomId} and emitting room_created socket event`)
      socket.join(roomId)
      socket.emit('room_created', roomId)
    } else if (numberOfClients == 1) {
      console.log(`Joining room ${roomId} and emitting room_joined socket event`)
      socket.join(roomId)
      socket.emit('room_joined', roomId)
    } else {
      console.log(`Can't join room ${roomId}, emitting full_room socket event`)
      socket.emit('full_room', roomId)
    }
  })
 
  // These events are emitted to all the sockets connected to the same room except the sender.
  socket.on('start_call', (roomId) => {
    console.log(`Broadcasting start_call event to peers in room ${roomId}`)
    socket.broadcast.to(roomId).emit('start_call')
  })
  socket.on('webrtc_offer', (event) => {
    console.log(`Broadcasting webrtc_offer event to peers in room ${event.roomId}`)
    socket.broadcast.to(event.roomId).emit('webrtc_offer', event.sdp)
  })
  socket.on('webrtc_answer', (event) => {
    console.log(`Broadcasting webrtc_answer event to peers in room ${event.roomId}`)
    socket.broadcast.to(event.roomId).emit('webrtc_answer', event.sdp)
  })
  socket.on('webrtc_ice_candidate', (event) => {
    console.log(`Broadcasting webrtc_ice_candidate event to peers in room ${event.roomId}`)
    socket.broadcast.to(event.roomId).emit('webrtc_ice_candidate', event)
  })
})
 
// START THE SERVER =================================================================
const port = process.env.PORT || 1483
server.listen(port, () => {
  console.log(`Express server listening on port ${port}`)
})
3. Tạo client views

Chúng ta sẽ tạo các chế độ xem của ứng dụng của chúng ta trong folder public / index.html. Ví dụ: một cái gì đó đơn giản sẽ hoạt động, chúng tôi sẽ sử dụng hai vùng chứa phần, một cho lựa chọn phòng và một hộp khác cho view hiển thị video. Lưu ý rằng chúng ta đang thêm kiểu cho các chế độ xem này bằng CSS và import thư viện SocketIO tại đây:

<!DOCTYPE html>
<html lang=”en”>
  <head>
    <meta charset=”UTF-8” />
    <meta name=”viewport” content=”width=device-width, initial-scale=1.0” />
    <title>WebRTC</title>
 
    <style type=”text/css”>
      body {
        margin: 0;
        font-size: 20px;
      }
 
      .centered {
        position: absolute;
        top: 40%;
        left: 50%;
        transform: translate(-50%, -50%);
      }
 
      .video-position {
        position: absolute;
        top: 35%;
        left: 50%;
        transform: translate(-50%, -50%);
      }
 
      #video-chat-container {
        width: 100%;
        background-color: black;
      }
 
      #local-video {
        position: absolute;
        height: 30%;
        width: 30%;
        bottom: 0px;
        left: 0px;
      }
 
      #remote-video {
        height: 100%;
        width: 100%;
      }
    </style>
  </head>
 
  <body>
    <div id=”room-selection-container” class=”centered”>
      <h1>WebRTC video conference</h1>
      <label>Enter the number of the room you want to connect</label>
      <input id=”room-input” type=”text” />
      <button id=”connect-button”>CONNECT</button>
    </div>
 
    <div id=”video-chat-container” class=”video-position” style=”display: none”>
      <video id=”local-video” autoplay=”autoplay”></video>
      <video id=”remote-video” autoplay=”autoplay”></video>
    </div>
 
    <script src=”/socket.io/socket.io.js”></script>
    <script type=”text/javascript” src=”client.js”></script>
  </body>
</html>
Bây giờ thì chạy máy chủ trên console và kiểm tra xem nó có đang chạy chính xác trong trình duyệt của bạn hay không và chế độ xem ứng dụng khách được hiển thị:

node server.js

localhost:1483


Thực hiện giao tiếp với client
Chúng ta sẽ thêm các chức năng cần thiết để ứng dụng hoạt động vào tệp public / client.js.

Đầu tiên, đây là cách client sẽ tham gia một phòng (hoặc tạo phòng nếu chưa có ai):


// DOM elements.
const roomSelectionContainer = document.getElementById('room-selection-container')
const roomInput = document.getElementById('room-input')
const connectButton = document.getElementById('connect-button')
 
const videoChatContainer = document.getElementById('video-chat-container')
const localVideoComponent = document.getElementById('local-video')
const remoteVideoComponent = document.getElementById('remote-video')
 
// Variables.
const socket = io()
const mediaConstraints = {
  audio: true,
  video: { width: 1280, height: 720 },
}
let localStream
let remoteStream
let isRoomCreator
let rtcPeerConnection // Connection between the local device and the remote peer.
let roomId
 
// Free public STUN servers provided by Google.
const iceServers = {
  iceServers: [
    { urls: 'stun:stun.l.google.com:19302' },
    { urls: 'stun:stun1.l.google.com:19302' },
    { urls: 'stun:stun2.l.google.com:19302' },
    { urls: 'stun:stun3.l.google.com:19302' },
    { urls: 'stun:stun4.l.google.com:19302' },
  ],
}
 
// BUTTON LISTENER ============================================================
connectButton.addEventListener('click', () => {
  joinRoom(roomInput.value)
})
 
// SOCKET EVENT CALLBACKS =====================================================
socket.on('room_created', async () => {
  console.log('Socket event callback: room_created')
 
  await setLocalStream(mediaConstraints)
  isRoomCreator = true
})
 
socket.on('room_joined', async () => {
  console.log('Socket event callback: room_joined')
 
  await setLocalStream(mediaConstraints)
  socket.emit('start_call', roomId)
})
 
socket.on('full_room', () => {
  console.log('Socket event callback: full_room')
 
  alert('The room is full, please try another one')
})
 
// FUNCTIONS ==================================================================
function joinRoom(room) {
  if (room === '') {
    alert('Please type a room ID')
  } else {
    roomId = room
    socket.emit('join', room)
    showVideoConference()
  }
}
 
function showVideoConference() {
  roomSelectionContainer.style = 'display: none'
  videoChatContainer.style = 'display: block'
}
 
async function setLocalStream(mediaConstraints) {
  let stream
  try {
    stream = await navigator.mediaDevices.getUserMedia(mediaConstraints)
  } catch (error) {
    console.error('Could not get user media', error)
  }
 
  localStream = stream
  localVideoComponent.srcObject = stream
}
Như bạn có thể thấy, chúng ta đang gọi phương thức Navigator.mediaDevices.getUserMedia để lấy dữ liệu media data của client. Nếu một client tham gia vào một phòng đã được tạo bởi một client khác, việc trao đổi phương tiện giữa các peers sẽ bắt đầu và sẽ được quản lý bởi các hàm và lệnh gọi lại sự kiện socket sau:


// SOCKET EVENT CALLBACKS =====================================================
socket.on('start_call', async () => {
  console.log('Socket event callback: start_call')
 
  if (isRoomCreator) {
    rtcPeerConnection = new RTCPeerConnection(iceServers)
    addLocalTracks(rtcPeerConnection)
    rtcPeerConnection.ontrack = setRemoteStream
    rtcPeerConnection.onicecandidate = sendIceCandidate
    await createOffer(rtcPeerConnection)
  }
})
 
socket.on('webrtc_offer', async (event) => {
  console.log('Socket event callback: webrtc_offer')
 
  if (!isRoomCreator) {
    rtcPeerConnection = new RTCPeerConnection(iceServers)
    addLocalTracks(rtcPeerConnection)
    rtcPeerConnection.ontrack = setRemoteStream
    rtcPeerConnection.onicecandidate = sendIceCandidate
    rtcPeerConnection.setRemoteDescription(new RTCSessionDescription(event))
    await createAnswer(rtcPeerConnection)
  }
})
 
socket.on('webrtc_answer', (event) => {
  console.log('Socket event callback: webrtc_answer')
 
  rtcPeerConnection.setRemoteDescription(new RTCSessionDescription(event))
})
 
socket.on('webrtc_ice_candidate', (event) => {
  console.log('Socket event callback: webrtc_ice_candidate')
 
  // ICE candidate configuration.
  var candidate = new RTCIceCandidate({
    sdpMLineIndex: event.label,
    candidate: event.candidate,
  })
  rtcPeerConnection.addIceCandidate(candidate)
})
 
// FUNCTIONS ==================================================================
function addLocalTracks(rtcPeerConnection) {
  localStream.getTracks().forEach((track) => {
    rtcPeerConnection.addTrack(track, localStream)
  })
}
 
async function createOffer(rtcPeerConnection) {
  let sessionDescription
  try {
    sessionDescription = await rtcPeerConnection.createOffer()
    rtcPeerConnection.setLocalDescription(sessionDescription)
  } catch (error) {
    console.error(error)
  }
 
  socket.emit('webrtc_offer', {
    type: 'webrtc_offer',
    sdp: sessionDescription,
    roomId,
  })
}
 
async function createAnswer(rtcPeerConnection) {
  let sessionDescription
  try {
    sessionDescription = await rtcPeerConnection.createAnswer()
    rtcPeerConnection.setLocalDescription(sessionDescription)
  } catch (error) {
    console.error(error)
  }
 
  socket.emit('webrtc_answer', {
    type: 'webrtc_answer',
    sdp: sessionDescription,
    roomId,
  })
}
 
function setRemoteStream(event) {
  remoteVideoComponent.srcObject = event.streams[0]
  remoteStream = event.stream
}
 
function sendIceCandidate(event) {
  if (event.candidate) {
    socket.emit('webrtc_ice_candidate', {
      roomId,
      label: event.candidate.sdpMLineIndex,
      candidate: event.candidate.candidate,
    })
  }
}
Bây giờ cả hai máy clients sẽ được kết nối, họ sẽ có thể nghe và nhìn thấy nhau bằng WebRTC và máy chủ sẽ không xử lý bất kỳ dữ liệu nào, nó chỉ được sử dụng để báo hiệu mặc dù chúng tôi vẫn có thể sử dụng các signalling để kết thúc cuộc gọi, kiểm soát kết nối hoặc bất kỳ mục đích nào khác.

Dưới đây là một số lời khuyên trước khi bạn chạy ứng dụng:

Sử dụng ngrok (hoặc dịch vụ tạo link publish) để hiển thị máy chủ localhost của bạn với một URL công khai và thử nghiệm ứng dụng với hai thiết bị khác nhau. Tuy nhiên, hãy cân nhắc rằng nó có thể gây ra một số chậm trễ giao tiếp.
Sử dụng tai nghe, phản hồi âm thanh có thể làm hỏng thính giác và thiết bị của bạn.
Kiểm tra xem WebRTC có được trình duyệt của bạn hỗ trợ hay không và đảm bảo rằng bạn sử dụng HTTPS, nếu không getUserMedia sẽ không hoạt động.
