---
layout: "post"
title: "[TIL] Android - MMS데이터 가져오기."
subtitle: "MMS데이터 가져오기."
date: 2023-04-10
author: "raindragonn"
lang: ko
categories: [TIL - Android]
tags:
    - MMS
    - ContentResolver
---

## MMS 데이터 가져오기.

- SMS는 비교적 간단하게 가져올 수 있지만 MMS는 여러 테이블을 통해 가져와야합니다.
- 아래 예제는 Mms에서도 텍스트로 이루어진 MMS만을 가져온다.

### 필요 권한

- `Manifest.permission.READ_SMS`

### 순서

1. TelePhony.Mms.Content_URI를 통해 id 및 날짜를 가져온다.
2. 1에서 가져온 id를 통해 “content://mms/$id/addr” Uri를 쿼리, 타입을 통해 보낸사람 번호를 가져온다.
3. 1에서 가져온 id를 통해 “content://mms/part” Uri를 쿼리, 텍스트로 된 Mms 데이터를 가져온다.

### 코드

```kotlin
class TestQueryHelper(
    private val _contentResolver: ContentResolver
) : QueryHelper<TestData> {

    private val mmsUri: Uri = Telephony.Mms.CONTENT_URI
    private val cursor: Cursor?
        get() =
            _contentResolver.query(
                mmsUri,
                null,
                null,
                null,
                null
            )

    override fun dataMapper(cursor: Cursor): TestData {
        val id = cursor.getInt(cursor.getColumnIndexOrThrow(Telephony.Mms._ID))
        val date = cursor.getLong(cursor.getColumnIndexOrThrow(Telephony.Mms.DATE)) * 1000
        val sender : String = getSender(id) ?: throw NullPointerException("NotFound Sender For Mms")
        val body: String = getBody(id) ?: throw NullPointerException("NotFound Body by Mms")
        
        return TestData(
            id = id,
            date = date,
            body = body,
            sender = sender
        )
    }

    private fun getSender(id: Int): String? {
        _contentResolver.query(
            Uri.parse("content://mms/$id/addr"),
            null,
            "${Telephony.Mms.Addr.MSG_ID} = ?",
            arrayOf(id.toString()),
            null
        )?.use { senderAddressCursor ->
            if (senderAddressCursor.moveToFirst()) {
                do {
                    // 137 수신, 151 발신 타입 값이다. Telephony.Mms.Addr.Type 의 주석 참고.
                    val isSender = senderAddressCursor.getInt(
                        senderAddressCursor.getColumnIndexOrThrow(Telephony.Mms.Addr.TYPE)
                    ) == 137
                    if (isSender) {
                        return senderAddressCursor.getString(
                            senderAddressCursor.getColumnIndexOrThrow(Telephony.Mms.Addr.ADDRESS)
                        )
                    }
                } while (senderAddressCursor.moveToNext())
            }
        }
        return null
    }

    private fun getBody(id: Int): String? {
        _contentResolver.query(
            Uri.parse("content://mms/part"),
            null,
            "${Telephony.Mms.Part.MSG_ID} = ?",
            arrayOf(id.toString()),
            null
        )?.use { partsCursor ->
            if (partsCursor.moveToFirst()) {
                do {
                    val partContentType =
                        partsCursor.getString(partsCursor.getColumnIndexOrThrow(Telephony.Mms.Part.CONTENT_TYPE))
                    if (partContentType == "text/plain") {
                        return partsCursor.getString(partsCursor.getColumnIndexOrThrow(Telephony.Mms.Part.TEXT))
                    }
                } while (partsCursor.moveToNext())
            }
        }
        return null
    }

    private fun convertData(cursor: Cursor?): List<TestData> {
        cursor ?: return emptyList()
        val result = mutableListOf<TestData>()
        cursor.use {
            if (cursor.moveToFirst()) {
                do {
                    val data = kotlin.runCatching {
                        dataMapper(cursor)
                    }.getOrNull()
                    data?.let { result.add(it) }
                } while (cursor.moveToNext())
            }
        }
        return result
    }

    override fun query(): List<TestData> {
        return convertData(cursor)
    }
}

interface QueryHelper<Data> {
    fun query(): List<Data>
    fun dataMapper(cursor: Cursor): Data
}

data class TestData(
    val id: Int,
    val date: Long,
    val body: String,
    val sender: String,
)
```

### 참고 자료

---

[https://developer.android.com/guide/topics/providers/content-providers](https://developer.android.com/guide/topics/providers/content-providers)

[https://stackoverflow.com/questions/3012287/how-to-read-mms-data-in-android](https://stackoverflow.com/questions/3012287/how-to-read-mms-data-in-android)