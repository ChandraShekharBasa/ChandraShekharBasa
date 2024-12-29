Locate the vulnerability :: Injection Flaws - HTTP Injection

'use strict'

const https = require('https')
const net = require('net')
const express = require('express')
const path = require('path')
const morgan = require('morgan')
const crypto = require('crypto')
const helmet = require('helmet')
const cluster = require('cluster')

const app = express()
const bodyParser = require('body-parser')
const cookieParser = require('cookie-parser')
const redis = require('redis')
const session = require('express-session')
const RedisStore = require('connect-redis')(session)
const config = require('./config.js')

const redisClient = redis.createClient({
    host: config.redisHost,
    db: config.redisSession.db,
})

const server = https.createServer({
    key: config.key,
    cert: config.cert,
}, app)

const logger = require('./logger/logger-init')
const redisDb = require('./redisDb/redisDb.js')
const mongoDb = require('./mongoDb/mongoDb.js')
const { checkNecessaryEnvVars } = require('./utils')

const envArr = [
    'GMAIL_USER',
    'GMAIL_PASSWORD',
]

function startApp() {
    checkNecessaryEnvVars(envArr, () => {
        mongoDb.initiate()
        .then(() => { redisDb.initiate() })
        .then(() => {
            app.use(helmet({
                frameguard: { action: 'deny' }
            }))
            app.disable('x-powered-by')
            const userDataRouts = require('./routes/usersData')
            const tasksRouts = require('./routes/tasks')

            app.set('views', path.join(__dirname, 'views'))
            app.set('view engine', 'pug')
            app.use(session({
                name: 'reminderapp',
                store: new RedisStore({
                    client: redisClient,
                }),
                resave: config.redisSession.resave,
                secret: crypto.randomBytes(
                    config.redisSession.secretBytesLength
                ).toString('base64')
                    || config.redisSession.secret,
                saveUninitialized: config.redisSession.saveUninitialized,
                httpOnly: config.redisSession.httpOnly,
                secure: config.redisSession.secure,
            }))


            app.use(morgan('dev'))
            app.use(bodyParser.json({ limit: '300mb', extended: true }))
            app.use(cookieParser())
            app.use(bodyParser.urlencoded({ extended: true }))

            app.use(express.static(path.join(__dirname, '../styles')))
            app.use(express.static(path.join(__dirname, '../scripts')))

            server.listen(config.port, () => {
                logger.info(`server running at port ${config.port}`)

                app.use(userDataRouts)
                app.use(tasksRouts)

                app.use((req, res) => {
                    res.status(404).send('Page Not Found')
                })
                app.use((err, req, res, next) => {
                    if (res.headersSent) {
                        return next(err)
                    }
                    if (err.message === 'invalid csrf token') {
                        return res.status(401).json({
                            error: 'invalid request credentials',
                            result: null
                        })
                    }
                    res.status(500)
                    res.json({
                        error: 'Server error',
                        result: null
                    })
                })
            })
        })
        .catch((error) => {
            logger.error(error)
            process.exit(1)
        })
    })
}

const netRedir = net.createServer((socket) => {
    socket.on('data', (data) => {
        const strings = data.toString().split('\n')
        const locationStr = strings[0].split(' ')[1]
        const regexp = /^\/verifymyemail\/.+/gi
        const isMatch = locationStr.match(regexp)
        if (isMatch) {
            const responceString = `HTTP/1.1 302 Found
            \nConnection: keep-alive
            \nLocation: http://127.0.0.1:1236`.concat(locationStr)
            socket.write(responceString)
            return socket.end()
        }
        return socket.end()
    })
})

function startRedirectApp() {
    netRedir.listen(config.redirectPort, () => {
        logger.info(`Redirector server running at port ${config.redirectPort}`)
    })
}

if (cluster.isMaster) {
    const { numWorkers } = config

    const redirectorId = numWorkers - 1
    let redirectorProcessId = -1

    logger.info(`Master cluster setting up ${numWorkers} workers...`)

    for (let i = 0; i < numWorkers; i += 1) {
        const e = { isRedirect: (i === redirectorId) }
        const w = cluster.fork(e)
        if (i === redirectorId) {
            redirectorProcessId = w.id
        }
    }

    cluster.on('online', (worker) => {
        logger.info(`Worker ${worker.process.pid} is online`)
    })

    cluster.on('exit', (worker, code, signal) => {
        const wasRedirector = (worker.process.pid === redirectorProcessId)

        logger.error(`Worker ${worker.process.pid}
            died with code: ${code},
            and signal:${signal}`)
        logger.info(`Starting a new worker, redirector:${wasRedirector}`)

        const w = cluster.fork({ isRedirect: wasRedirector })
        if (wasRedirector) {
            redirectorProcessId = w.pid
        }
    })
} else {
    logger.info(`worker ${process.pid} is redirector:${process.env.isRedirect}`)

    if (process.env.isRedirect === 'true') {
        startRedirectApp()
    } else {
        startApp()
    }
}
