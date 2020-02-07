const yaml = require('js-yaml')
const noOrgConfig = false

class OpenIssueOnRepoCreation {

  // Analyze checks for the existence of an organization-wide repository created for the purpose of configurng Probot apps. By default, the path of this configuration file is https://github.com/[ORG_NAME]/org-settings/.github/create-email-notifications.yml. Both the repository name and file path can be configured in defaults.js. If no configuration file exists at the specified location, default values are used as configured in defaults.js

  static analyze (github, repo, payload, logger) {
    const defaults = require('./defaults')
    const orgRepo = (process.env.ORG_WIDE_REPO_NAME) ? process.env.ORG_WIDE_REPO_NAME : defaults.ORG_WIDE_REPO_NAME
    const filename = (process.env.FILE_NAME) ? process.env.FILE_NAME : defaults.FILE_NAME
    logger.info('Get config from: ' + repo.owner + '/' + orgRepo + '/' + filename)

    return github.repos.getContent({
      owner: repo.owner,
      repo: orgRepo,
      path: filename
    }).catch(() => ({
      noOrgConfig
    }))
      .then((orgConfig) => {
        if ('noOrgConfig' in orgConfig) {
          logger.info('NOTE: config file not found in: ' + orgRepo + '/' + filename + ', using defaults.')
          return new OpenIssueOnRepoCreation(github, repo, payload, logger, '').openIssue()
        } else {
          const content = Buffer.from(orgConfig.data.content, 'base64').toString()
          return new OpenIssueOnRepoCreation(github, repo, payload, logger, content).openIssue()
        }
      })
  }

  constructor (github, repo, payload, logger, config) {
    this.github = github
    this.repo = repo
    this.payload = payload
    this.logger = logger
    this.config = yaml.safeLoad(config)
  }

  openIssue () {
    var configParams = Object.assign({}, require('./defaults'), this.config || {})

    // Ensures that enableIssueCreation = true in the configuration. Otherwise, it logs the skip and exits
    if (!configParams.enableIssueCreation) {
        this.logger.info('Repo: ' + this.repo.repo + ' was created but enableIssueCreation is set to false')
        return
    }

    var issueBody = formIssueBody(this.payload, configParams.createdIssueBody, configParams.ccList)

    this.logger.info('Creating repo!')

    createIssue(this.repo, this.github, configParams.createdIssueTitle, issueBody)

  }

}

function createIssue(repo, github, title, body) {
  const issueParams = {
    title: title,
    body: body
  }
  const createIssueParams = Object.assign({}, repo, issueParams || {})
  github.issues.create(createIssueParams)
}

function formIssueBody(payload, body, ccList) {
  const owner = payload.sender.login
  var issueBody = body + '\n\n/cc @' + owner
  issueBody += (ccList) ? '\n/cc ' + ccList : ''
  return issueBody
}



module.exports = OpenIssueOnRepoCreation
