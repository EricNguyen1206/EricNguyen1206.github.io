---
layout: post
title: "Audit Log trong hệ thống phần mềm: từ khái niệm đến implement với Node.js + Pino"
date: 2026-08-23 15:00:00 +0700
categories: programming
---

Hôm nay mình sẽ nói về một thứ mà ai làm hệ thống "nghiêm túc" cũng sẽ bị
đòi hỏi sớm muộn: **audit log** (hay audit trail). Kiến thức này nằm rải rác
khắp nơi, mình xin gom lại từ A đến Z, cuối bài thì code thật bằng Node.js
với thư viện `pino`.

## Vấn đề

Tưởng tượng bạn quản lý một cửa hàng. Một buổi sáng đẹp trời, khách gọi lên
kêu: *"Tôi đặt cái PS5 hôm qua mà giờ đơn biến mất mất rồi!"*

Bạn hỏi nhân viên A — A nói không đụng vào. Hỏi nhân viên B — B nói chỉ sửa tên khách. Vậy ai đã xoá đơn? Xoá lúc mấy giờ? Trước khi xoá nó trông thế nào?

Không có ghi chép gì thì chuyện này thành **treo máy**. Mọi người đều vô tội, và bạn thì không sửa được gì.

## Giải pháp: ghi lại hết

Audit log chính là quyển "sổ nhật ký" của hệ thống:

> **Mỗi khi có ai đó làm gì đó với dữ liệu, ghi lại một dòng: ai, làm gì,
> với đối tượng nào, lúc mấy giờ, dữ liệu trước và sau khi thay đổi.**

Cửa hàng trên có sổ này thì chỉ cần lật trang: *"14:32 — nhân viên B — xoá đơn #1234"* — xong, không cãi nhau được.

Các ví dụ đời thực khác cùng bản chất:

- **Sổ kế toán**: mỗi phiếu thu chi đều có chứng từ, muốn sửa phải ghi đè bằng bút toán đối ngẫu — không được xoá.
- **Hộ chiếu**: mỗi lần bạn xuất nhập cảnh, đóng một cái dấu. Lịch sử đi lại không thể chỉnh sửa.
- **Camera an ninh**: quay lại mọi thứ, ai vào nhà lúc nào.

Điểm chung: **chỉ được ghi thêm, không được sửa xoá** (append-only).

## Audit log khác gì application log?

Nhiều người nhầm hai cái này là một. Để hiểu ngắn gọn:

|          | Application log                    | Audit log                            |
| -------- | ---------------------------------- | ------------------------------------ |
| Mục đích | Debug, theo dõi sức khoẻ hệ thống  | Truy trách nhiệm, chứng minh         |
| Nội dung | Exception, request chậm, memory... | Hành động của user lên dữ liệu       |
| Ai đọc   | Dev, SRE                           | Kiểm toán, pháp chế, sếp, khách hàng |
| Bắt buộc | Tùy                                | Có khi theo luật (y tế, tài chính)   |

Một dòng `"DB connection timeout"` là application log. Một dòng
`"user 42 đổi giá đơn hàng #99 từ 1.000đ thành 10.000.000đ"` là audit log.

Cùng có thể dùng `pino` để ghi cả hai, nhưng nên **tách file / tách đích
lưu trữ** — ta sẽ làm vậy ở phần code.

## Một entry audit log chuẩn gồm gì?

Hãy nhớ công thức 4W1H:

* **Who**: Ai thực hiện?
* **What**: Làm gì?
* **Where**: Ở đâu?
* **When**: Khi nào?
* **How**: Như thế nào?

ví dụ

```json
{
  "created_at": "2026-08-23T10:15:30.000Z", // when
  "actor": { "user_id": 42, "username": "eric", "ip": "203.0.113.7" }, // who
  "action": "order.update", // what
  "resource": { "type": "order", "id": 99 }, // where
  "changes": {
    "price": { "from": 1000, "to": 10000000 },
    "status": { "from": "draft", "to": "confirmed" }
  },
  "result": "success"
}
```

Đủ để trả lời mọi câu hỏi: ai (who), làm gì (what), với cái gì (target),
khi nào (when), ở đâu/từ IP nào (where), và dữ liệu đổi ra sao (changes).

**Lưu ý vàng:** đừng bao giờ ghi mật khẩu, token, số thẻ, CCCD... vào
audit log. Ghi `{"password": "***"}` thay vì giá trị thật. Log bị leak là
chuyện thường thấy ở tướng.

## Các mức độ "xịn" khi implement

Từ thấp đến cao:

1. **Ghi chung vào application log** — nhanh gọn, nhưng khó truy vấn, dễ bị
   trôi theo log rotation.
2. **File riêng `audit.log` (append-only)** — mức ta sẽ code hôm nay bằng
   pino. Đủ tốt cho hệ thống vừa và nhỏ.
3. **Bảng riêng trong DB** (`audit_logs`) — dễ JOIN, dễ query, có transaction.
   Nhược: ai có quyền DB thì sửa/xoá được — phải chặn.
4. **Append-only / event sourcing** — lưu toàn bộ dòng sự kiện, trạng thái
   hiện tại là kết quả replay. Xịn nhất, tốn công nhất.
5. **Dịch vụ chuyên dụng** — AWS CloudTrail, hoặc dịch vụ audit riêng.

Bài này dừng ở mức 2, nhưng biết đường đi của mức 3–4 để sau này nâng cấp.

## Implement với Node.js + Pino

### Vì sao pino?

`pino` là logger nhanh nhất thế giới Node hiện nay (JSON, async I/O).
Nhanh quan trọng với audit log vì mỗi request có thể sinh nhiều dòng.

### Setup dự án

```sh
npm install pino pino-http express
```

### Tạo audit logger riêng

Điểm mấu chốt: audit logger **tách biệt** app logger, ghi ra file riêng,
level `info` trở lên, format JSON:

```js
// audit-logger.js
const pino = require('pino');

const auditLogger = pino(
  {
    level: 'info',
    base: null, // bỏ các field mặc định của pino, giữ entry sạch
    timestamp: pino.stdTimeFunctions.isoTime, // when: ISO 8601
    redact: {
      paths: [
        'changes.password', 'changes.token', 'changes.*.from.password',
      ],
      censor: '[REDACTED]',
    },
  },
  pino.destination({ dest: 'logs/audit.log', sync: false })
);

module.exports = auditLogger;
```

Giải thích vài dòng khó:

- `pino.destination({ sync: false })` — ghi bất đồng bộ, cực nhanh, không
  block request. Nghe có vẻ rủi ro mất log khi crash, nhưng với audit
  thường thì chấp nhận được (mức xịn hơn sẽ ghi vào DB/Kafka).
- `redact` —ẩn trường nhạy cảm trước khi ghi, thứ bắt buộc phải có.
- `isoTime` — thời gian chuẩn ISO 8601, kèm timezone, khỏi cãi nhau
  "giờ máy chủ hay giờ khách".

### Helper để ghi có cấu trúc

Đừng gọi lẻ tẻ `auditLogger.info(...)` khắp codebase. Viết một hàm
gạch đầu dòng cho cả team xài chung:

```js
// audit.js
const auditLogger = require('./audit-logger');

async function audit({ actor, action, target, req, changes, result }) {
  auditLogger.info({
    who: {
      userId: actor?.id,
      username: actor?.username,
      ip: req?.ip,
      userAgent: req?.headers?.['user-agent'],
    },
    what: action,             // VD: 'order.update'
    where: req?.originalUrl,
    target,                   // { type: 'order', id: 99 }
    changes,                  // { price: { from: 1, to: 2 } }
    result: result ?? 'success',
  }, action);
}

module.exports = { audit };
```

### Dùng trong route Express

Giờ trong handler, mỗi thao tác quan trọng ta gọi một phát:

```js
// routes/orders.js
const express = require('express');
const { audit } = require('../audit');

const router = express.Router();

router.patch('/orders/:id', async (req, res) => {
  const order = await db.orders.findById(req.params.id);
  const before = { price: order.price, status: order.status };

  order.price = req.body.price ?? order.price;
  order.status = req.body.status ?? order.status;
  await order.save();

  const changes = diff(before, {
    price: order.price,
    status: order.status,
  });

  await audit({
    actor: req.user,                       // từ auth middleware
    action: 'order.update',
    target: { type: 'order', id: order.id },
    req,
    changes,
  });

  res.json(order);
});

// Hàm so sánh cũ → mới, chỉ trả về field realmente thay đổi
function diff(before, after) {
  const changes = {};
  for (const key of Object.keys(after)) {
    if (before[key] !== after[key]) {
      changes[key] = { from: before[key], to: after[key] };
    }
  }
  return changes;
}

module.exports = router;
```

Chạy thử rồi `cat logs/audit.log`, bạn sẽ thấy dòng kiểu:

```json
{"level":30,"time":"2026-08-23T08:15:30.000Z","who":{"userId":42,"username":"eric","ip":"::1"},"what":"order.update","where":"/orders/99","target":{"type":"order","id":99},"changes":{"price":{"from":1000,"to":10000000}},"result":"success"}
```

Đúng chuẩn 5W ở trên — đọc là hiểu liền, không cần giải thích.

### Bonus: audit mọi request bằng middleware

Với thao tác nhạy cảm mà bạn muốn ghi **cả khi thất bại** (đăng nhập sai,
không có quyền...), middleware là chỗ gọn nhất:

```js
// audit-middleware.js
const auditLogger = require('./audit-logger');

function auditSensitiveActions(actions = ['POST', 'PUT', 'PATCH', 'DELETE']) {
  return (req, res, next) => {
    if (!actions.includes(req.method)) return next();
    res.on('finish', () => {
      if (res.statusCode >= 400) {
        auditLogger.warn({
          who: { userId: req.user?.id, username: req.user?.username, ip: req.ip },
          what: `${req.method.toLowerCase()} ${req.originalUrl}`,
          result: 'failure',
          statusCode: res.statusCode,
        });
      }
    });
    next();
  };
}

module.exports = auditSensitiveActions;
```

## Ưu — nhược điểm của cách này

**Ưu:**

- Cài trong 30 phút, không cần hạ tầng mới.
- pino nhanh, JSON structured dễ đưa vào ELK/Loki/CloudWatch để truy vấn.
- Tách file riêng nên không lẫn với log debug.

**Nhược:**

- File nằm trên disk máy chủ — cần backup + rotation (`logrotate` hoặc
  `pino.transport` với rolling).
- Ai có quyền SSH vào máy là sửa/xoá được log. Mức "chống chọi" thấp hơn
  ghi vào DB append-only hoặc gửi sang hệ thống bên ngoài.
- Không có transaction với nghiệp vụ: code chạy xong nhưng crash trước khi
  ghi log thì mất dấu vết.

## Khi nào thì cần hơn pino?

- **Cần query "ai đã đụng vào bản ghi X" thường xuyên** → làm bảng
  `audit_logs` trong DB, index theo `target_type + target_id`.
- **Compliance nghiêm ngặt** (y tế, tài chính, GDPR) → append-only storage,
  hash chain (mỗi entry chứa hash của entry trước — sửa một dòng là bể cả
  chuỗi), hoặc dịch vụ chuyên dụng.
- **Muốn replay dữ liệu** → đọc về event sourcing, một chơi khác hẳn.

## Tóm tắt

- Audit log = sổ nhật ký **ai đã làm gì với dữ liệu, khi nào** — phục vụ
  truy trách nhiệm và chứng minh, khác với log debug.
- Entry chuẩn cần đủ: **who, what, when, where, target, changes, result**.
- Với Node.js: pino + file riêng + hàm `audit()` thống nhất + redact dữ
  liệu nhạy cảm.
- Append-only là linh hồn của audit log: **không sửa, không xoá**.
- Hệ thống lớn hơn thì nâng cấp dần sang DB table hoặc event store.

Checklist nhanh khi tự review:

- [ ] Mọi thao tác ghi/sửa/xoá đều có audit?
- [ ] Có ghi cả thao tác **thất bại** (đăng nhập sai, từ chối quyền)?
- [ ] Dữ liệu nhạy cảm đã bị redact?
- [ ] Log có thời gian ISO + timezone rõ ràng?
- [ ] File log có rotation và backup?
- [ ] Log append-only về mặt quyền truy cập?

Chúc bạn không bao giờ phải Treo máy kiểu "ai xoá đơn của tôi?" nữa.
