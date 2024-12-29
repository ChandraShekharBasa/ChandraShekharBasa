const netRedir = net.createServer((socket) => {
    socket.on('data', (data) => {
        const strings = data.toString().split('\n')
        const locationStr = strings[0].split(' ')[1]
        const regexp = /^\/verifymyemail\/.+/gi
        const isMatch = locationStr.match(regexp)
        if (isMatch) {
            const responceString = `HTTP/1.1 302 Found
            \nConnection: keep-alive
            \nX-Frame-Options: DENY
            \nX-Download-Options: noopen
            \nLocation: http://127.0.0.1:1236`.concat(locationStr)
            socket.write(responceString)
            return socket.end()
        }
        return socket.end()
    })
})


const netRedir = net.createServer((socket) => {
    socket.on('data', (data) => {
        const strings = data.toString().split('\n')
        const locationStr = strings[0].split(' ')[1]
        const regexp = /^\/verifymyemail\/.+/gi
        const isMatch = locationStr.match(regexp)
        if (isMatch) {
            const responceString = `HTTP/1.1 302 Found
            \nConnection: keep-alive
            \nLocation: http://127.0.0.1:1236%locationStr%`
                .replace('locationStr', locationStr)
            socket.write(responceString)
            return socket.end()
        }
        return socket.end()
    })
})


const netRedir = net.createServer((socket) => {
    socket.on('data', (data) => {
        const strings = data.toString().split('\n')
        const locationStr = strings[0].split(' ')[1]
        const regexp = /^\/verifymyemail\/.+/gi
        const isMatch = locationStr.match(regexp)
        if (isMatch) {
            const responceString = `HTTP/1.1 302 Found
            \nLocation: http://127.0.0.1:1236`
                .concat(locationStr)
                .concat('\nConnection: keep-alive')
            socket.write(responceString)
            return socket.end()
        }
        return socket.end()
    })
})
