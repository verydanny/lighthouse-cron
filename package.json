import * as path from 'path'
import mkdirp from 'mkdirp'
import lighthouse from 'lighthouse'
import * as chromeLauncher from 'chrome-launcher'
import log from 'lighthouse-logger'
import { saveAssets } from 'lighthouse/lighthouse-core/lib/asset-saver'
import { write } from 'lighthouse/lighthouse-cli/printer'
import cron from 'node-cron'

const runEveryThisMinutes = 30
const sampleSize = 5

const opts = {
  emulatedFormFactor: 'desktop',
  output: 'html',
  chromeFlags: ['--headless'],
  logLevel: 'info'
}

function launchChromeAndRunLighthouse(url, opts, config = null) {
  log.setLevel({ logLevel: opts.logLevel })

  return chromeLauncher
    .launch({ chromeFlags: opts.chromeFlags })
    .then(chrome => {
      opts.port = chrome.port

      return lighthouse(url, opts, config)
        .then(results => {
          return chrome.kill().then(() => results)
        })
        .catch(e => console.log(e))
    })
    .catch(e => console.log(e))
}

async function run(sampleSize) {
  const currentDate = new Date()
  const month = currentDate.getMonth() + 1
  const day = currentDate.getDate()
  const hourMinute = `${currentDate.getHours()}.${currentDate.getMinutes()}`
  const folderStructure = `./reports/${month}/${day}/${hourMinute}`

  for (let i = 1; i <= sampleSize; i++) {
    const resolvedPath = path.resolve(
      process.cwd(),
      folderStructure,
      `run-${i}`
    )
    const reportOutputPath = `${resolvedPath}/report.${opts.output}`

    try {
      const results = await launchChromeAndRunLighthouse(
        'https://www.apartmentguide.com/apartments/Georgia/Atlanta/',
        opts
      )
      const { lhr, artifacts, report } = results
      const reportOutput = report[opts.output.indexOf(opts.output)]

      await mkdirp(resolvedPath)
      await saveAssets(artifacts, lhr.audits, `${resolvedPath}/out`)
      await write(reportOutput, opts.output, reportOutputPath)
    } catch (e) {
      console.log(e)
    }
  }
}

const lighthouseCron = cron.schedule(
  `*/${runEveryThisMinutes.toString()}} * * * *`,
  async () => run(sampleSize)
)

process.on('SIGINT', () => {
  lighthouseCron.destroy()

  process.exit(1)
})
