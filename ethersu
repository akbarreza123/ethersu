const axios = require('axios');
const fs = require('fs');
const path = require('path');
const readline = require('readline');

// File paths
const hashFile = path.join(__dirname, 'hash.txt');
const freshTokenFile = path.join(__dirname, 'fresh-token.txt');
const tokenFile = path.join(__dirname, 'token.txt');

// Constants for delay
const delay = ms => new Promise(resolve => setTimeout(resolve, ms));
const delayTime = 600000; // 10 minutes delay

// Create readline interface for user input
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});

// Function to display header in console
function displayHeader() {
    console.log("===========================================");
    console.log("         🚀 Drops Bot by BOTERDROP 🚀       ");
    console.log("===========================================");
}

// Function to get token for each account
async function getToken(queryId) {
    const payload = {
        "webAppData": queryId
    };

    try {
        const response = await axios.post('https://api.miniapp.dropstab.com/api/auth/login', payload, {
            headers: {
                'Content-Type': 'application/json',
                'user-agent': 'Mozilla/5.0'
            }
        });

        const accessToken = response.data.jwt.access.token;
        const refreshToken = response.data.jwt.refresh.token;

        return { accessToken, refreshToken };
    } catch (error) {
        console.error(`❌ Error during login for queryId ${queryId}: ${error.message}`);
        return null;
    }
}

// Function to refresh token if expired
async function refreshToken(refreshToken) {
    const payload = {
        "refreshToken": refreshToken
    };

    try {
        const response = await axios.post('https://api.miniapp.dropstab.com/api/auth/refresh', payload, {
            headers: {
                'Content-Type': 'application/json',
                'user-agent': 'Mozilla/5.0'
            }
        });

        if (response.data && response.data.access && response.data.refresh) {
            const newAccessToken = response.data.access.token;
            const newRefreshToken = response.data.refresh.token;
            return { newAccessToken, newRefreshToken };
        } else {
            console.error('⚠️ Unexpected response structure when refreshing token:', response.data);
            return null;
        }
    } catch (error) {
        console.error(`❌ Error refreshing token: ${error.message}`);
        return null;
    }
}

// Function to claim welcome bonus
async function claimWelcomeBonus(token) {
    try {
        const response = await axios.post('https://api.miniapp.dropstab.com/api/bonus/welcomeBonus', {}, {
            headers: {
                'Authorization': `Bearer ${token}`,
                'Content-Type': 'application/json',
                'user-agent': 'Mozilla/5.0'
            }
        });
        console.log(`🎁 Welcome bonus claimed: Bonus = ${response.data.bonus}`);
    } catch (error) {
        console.error(`❌ Error claiming welcome bonus: ${error.message}`);
    }
}

// Function to fetch referral link
async function fetchReferralLink(token) {
    try {
        const response = await axios.get('https://api.miniapp.dropstab.com/api/refLink', {
            headers: {
                'Authorization': `Bearer ${token}`,
                'Content-Type': 'application/json',
                'user-agent': 'Mozilla/5.0'
            }
        });

        console.log(`🔗 Referral link details: Code = ${response.data.code}, Balance = ${response.data.balance}`);
    } catch (error) {
        console.error(`❌ Error fetching referral link: ${error.message}`);
    }
}

// Function to append tokens to a file without replacing existing ones
function appendTokensToFile(filePath, tokens) {
    const existingTokens = readTokensFromFile(filePath);
    const updatedTokens = [...existingTokens, ...tokens];
    fs.writeFileSync(filePath, updatedTokens.join('\n'), 'utf8');
}

// Function to replace tokens in a file (replace specific lines)
function replaceTokensInFile(filePath, tokens) {
    fs.writeFileSync(filePath, tokens.join('\n'), 'utf8');
}

// Function to read tokens from a file
function readTokensFromFile(filePath) {
    if (fs.existsSync(filePath)) {
        return fs.readFileSync(filePath, 'utf8').split('\n').filter(Boolean);
    }
    return [];
}

// Function to check if token is expired (based on expiration time in token)
function tokenExpired(token) {
    try {
        const payload = JSON.parse(Buffer.from(token.split('.')[1], 'base64').toString());
        const expirationTime = payload.exp * 1000; // Convert to milliseconds
        const currentTime = Date.now();
        return currentTime >= expirationTime; // Token is expired if current time is greater than or equal to expiration
    } catch (error) {
        console.error('⚠️ Error checking token expiration:', error.message);
        return true; // Assume token is expired if there's an error
    }
}

// Function to get tokens for all accounts (adds token to file without replacing)
async function getTokensForAllAccounts() {
    const accounts = fs.readFileSync(hashFile, 'utf8').split('\n').filter(Boolean);
    let accessTokens = [];
    let refreshTokens = [];

    // Process all queries in hash.txt
    for (let i = 0; i < accounts.length; i++) {
        const account = accounts[i];
        console.log(`🔑 Getting token for account with query_id: ${account}`);

        const tokens = await getToken(account);
        if (tokens) {
            accessTokens.push(tokens.accessToken); // Add new access token
            refreshTokens.push(tokens.refreshToken); // Add new refresh token
        } else {
            console.log(`⚠️ Failed to get token for account ${i + 1}`);
        }

        await delay(2000); // Small delay to avoid overwhelming server
    }

    // Append new tokens to respective files
    appendTokensToFile(freshTokenFile, accessTokens);
    appendTokensToFile(tokenFile, refreshTokens);

    console.log('✅ New tokens appended to fresh-token.txt and token.txt');
}

// Function to process accounts during run bot
async function processAccounts() {
    const accounts = fs.readFileSync(hashFile, 'utf8').split('\n').filter(Boolean);
    let accessTokens = readTokensFromFile(freshTokenFile);
    let refreshTokens = readTokensFromFile(tokenFile);

    for (let i = 0; i < accounts.length; i++) {
        let token = accessTokens[i];

        // Check if access token is expired
        if (tokenExpired(token)) {
            console.log(`⏳ Access token expired for account ${i + 1}. Refreshing token...`);

            const refreshedTokens = await refreshToken(refreshTokens[i]);
            if (refreshedTokens) {
                token = refreshedTokens.newAccessToken;
                accessTokens[i] = token; // Replace the expired access token in fresh-token.txt
                refreshTokens[i] = refreshedTokens.newRefreshToken; // Replace the used refresh token in token.txt
            } else {
                console.log(`⚠️ Failed to refresh token for account ${i + 1}. Skipping.`);
                continue;
            }
        }

        try {
            console.log(`⚙️ Processing Account ${i + 1}`);

            // Claim Welcome Bonus and fetch Referral Link at the start
            await claimWelcomeBonus(token);
            await fetchReferralLink(token);

            const userInfo = await getUserInfo(token); // Get user info and balance
            console.log(`👤 Username: ${userInfo.tgUsername}, 💰 Balance: ${userInfo.balance}`);

            await claimDailyBonus(token); // Claim daily bonus
            await processTasks(token); // Process tasks (directly verify and claim)
            const hasActiveTrade = await checkActiveTrade(token); // Check if there is an active trade
            if (!hasActiveTrade) {
                await performTrading(token); // Perform trading game actions only if no active trade
            } else {
                console.log(`⚖️ Account ${i + 1} already has active trade. Skipping trading.`);
            }

            // Handle 4-hour and 24-hour trading if balance allows unlock
            await processUnlockedPeriods(token, userInfo.balance);

            // Claim reward for any winning trades
            await checkAndClaimWinningTrades(token);

        } catch (error) {
            if (error.response && error.response.status === 401) {
                console.log(`❌ Error 401 for account ${i + 1}. Refreshing token...`);
                const newToken = await refreshToken(token);
                if (newToken) {
                    accessTokens[i] = newToken; // Replace expired token in fresh-token.txt
                    refreshTokens[i] = newToken; // Update refresh token
                    replaceTokensInFile(freshTokenFile, accessTokens); // Save the refreshed token to file
                    replaceTokensInFile(tokenFile, refreshTokens);
                    console.log(`✅ Token for account ${i + 1} refreshed.`);
                } else {
                    console.log(`⚠️ Failed to refresh token for account ${i + 1}.`);
                }
            } else {
                console.error(`❌ Error processing account ${i + 1}: ${error.message}`);
            }
        }

        await delay(10000); // 10 seconds delay between accounts to avoid overwhelming the server
    }

    // Save updated tokens
    replaceTokensInFile(freshTokenFile, accessTokens);
    replaceTokensInFile(tokenFile, refreshTokens);

    console.log('✅ All accounts processed. Waiting 10 minutes before the next cycle.');
    await delay(delayTime); // Wait 10 minutes before looping again

    // Looping back to process accounts again
    await processAccounts();
}

// Function to claim reward for winning trades
async function claimReward(orderId, token) {
    try {
        const response = await axios.put(`https://api.miniapp.dropstab.com/api/order/${orderId}/claim`, {}, {
            headers: {
                'Authorization': `Bearer ${token}`,
                'Content-Type': 'application/json',
                'user-agent': 'Mozilla/5.0'
            }
        });
        console.log(`🎉 Reward claimed for order ${orderId}. Response: `, response.data);
    } catch (error) {
        console.error(`❌ Error claiming reward for order ${orderId}: ${error.message}`);
    }
}

// Function to check and claim winning trades if the status is CLAIM_AVAILABLE
async function checkAndClaimWinningTrades(token) {
    try {
        const response = await axios.get('https://api.miniapp.dropstab.com/api/order', {
            headers: {
                'Authorization': `Bearer ${token}`,
                'Content-Type': 'application/json',
                'user-agent': 'Mozilla/5.0'
            }
        });

        const periods = response.data.periods;

        for (const period of periods) {
            if (period.order && period.order.status === 'CLAIM_AVAILABLE') {
                console.log(`Order ${period.order.id} is ready for claim. Claiming reward...`);
                await claimReward(period.order.id, token);
            } else {
                console.log(`Order ${period.order ? period.order.id : 'N/A'} is not ready for claim.`);
            }
        }
    } catch (error) {
        console.error('Error checking for winning trades: ', error.message);
    }
}

// Function to get user information and balance
async function getUserInfo(token) {
    try {
        const response = await axios.get('https://api.miniapp.dropstab.com/api/user/current', {
            headers: {
                'Authorization': `Bearer ${token}`,
                'Content-Type': 'application/json',
                'user-agent': 'Mozilla/5.0'
            }
        });

        return response.data; // Return user information (includes balance and username)
    } catch (error) {
        console.error('Error fetching user info: ', error.message);
        return null;
    }
}

// Function to perform daily bonus claim
async function claimDailyBonus(token) {
    try {
        const response = await axios.post('https://api.miniapp.dropstab.com/api/bonus/dailyBonus', {}, {
            headers: {
                'Authorization': `Bearer ${token}`,
                'Content-Type': 'application/json',
                'user-agent': 'Mozilla/5.0'
            }
        });
        console.log('Daily bonus claimed: ', response.data);
    } catch (error) {
        console.error('Error claiming daily bonus: ', error.message);
    }
}

// Function to process tasks (directly verify and claim all tasks)
async function processTasks(token) {
    try {
        // Get quest list
        const questResponse = await axios.get('https://api.miniapp.dropstab.com/api/quest', {
            headers: {
                'Authorization': `Bearer ${token}`,
                'Content-Type': 'application/json',
                'user-agent': 'Mozilla/5.0'
            }
        });

        const quests = questResponse.data;
        console.log('Quest list fetched. Verifying and claiming tasks if not completed...');

        // Verify and claim tasks that are not completed
        for (const questCategory of quests) {
            for (const quest of questCategory.quests) {
                if (quest.status !== 'COMPLETED') { // Only process tasks that are not COMPLETED
                    await verifyTask(quest.id, token); // Verify task
                    await claimTask(quest.id, token);  // Claim task
                } else {
                    console.log(`Task ${quest.id} is already COMPLETED. Skipping...`);
                }
            }
        }
    } catch (error) {
        console.error('Error fetching quests or processing tasks: ', error.message);
    }
}

// Function to claim tasks
async function claimTask(taskId, token) {
    try {
        const response = await axios.put(`https://api.miniapp.dropstab.com/api/quest/${taskId}/claim`, {}, {
            headers: {
                'Authorization': `Bearer ${token}`,
                'Content-Type': 'application/json',
                'user-agent': 'Mozilla/5.0'
            }
        });

        console.log(`Task ${taskId} claimed. Response: `, response.data);
    } catch (error) {
        console.error(`Error claiming task ${taskId}: `, error.message);
    }
}

// Function to verify tasks
async function verifyTask(taskId, token) {
    try {
        const response = await axios.put(`https://api.miniapp.dropstab.com/api/quest/${taskId}/verify`, {}, {
            headers: {
                'Authorization': `Bearer ${token}`,
                'Content-Type': 'application/json',
                'user-agent': 'Mozilla/5.0'
            }
        });

        console.log(`Task ${taskId} verified. Response: `, response.data);
    } catch (error) {
        console.error(`Error verifying task ${taskId}: `, error.message);
    }
}

// Function to check if there is an active trade
async function checkActiveTrade(token) {
    try {
        const response = await axios.get('https://api.miniapp.dropstab.com/api/order', {
            headers: {
                'Authorization': `Bearer ${token}`,
                'Content-Type': 'application/json',
                'user-agent': 'Mozilla/5.0'
            }
        });

        const periods = response.data.periods;
        // If any period has an order with a status of "PENDING", there is an active trade
        return periods.some(period => period.order && period.order.status === 'PENDING');
    } catch (error) {
        console.error('Error checking active trades: ', error.message);
        return false;
    }
}

// Function to process unlocked 1-hour, 4-hour, and 24-hour trading periods
async function processUnlockedPeriods(token, balance) {
    // Unlock thresholds (adjust based on the business rules)
    const unlockThreshold4Hours = 5000; // Example threshold for 4-hour period
    const unlockThreshold24Hours = 25000; // Example threshold for 24-hour period

    try {
        const response = await axios.get('https://api.miniapp.dropstab.com/api/order', {
            headers: {
                'Authorization': `Bearer ${token}`,
                'Content-Type': 'application/json',
                'user-agent': 'Mozilla/5.0'
            }
        });

        const periods = response.data.periods;

        // Handle the 1-hour period trading
        const oneHourPeriod = periods.find(period => period.period.hours === 1);
        if (oneHourPeriod && oneHourPeriod.order) {
            if (oneHourPeriod.order.status === "NOT_WIN") {
                console.log('1-hour trade lost. Retrying trading...');
                await performTrading(token, 1); // Re-trade for 1-hour period
            } else if (oneHourPeriod.order.status === "PENDING") {
                console.log('1-hour trade is still pending.');
            }
        } else if (!oneHourPeriod.order) {
            console.log('No active trade in 1-hour period. Starting new trade...');
            await performTrading(token, 1); // New trade for 1-hour period
        }

        // Handle the 4-hour period trading
        const fourHourPeriod = periods.find(period => period.period.hours === 4);
        if (fourHourPeriod && balance >= unlockThreshold4Hours) {
            if (fourHourPeriod.order) {
                if (fourHourPeriod.order.status === "NOT_WIN") {
                    console.log('4-hour trade lost. Retrying trading...');
                    await performTrading(token, 2); // Re-trade for 4-hour period
                } else {
                    console.log('4-hour trading is still in progress.');
                }
            } else {
                console.log('No active trade in 4-hour period. Starting new trade...');
                await performTrading(token, 2); // New trade for 4-hour period
            }
        }

        // Handle the 24-hour period trading
        const twentyFourHourPeriod = periods.find(period => period.period.hours === 24);
        if (twentyFourHourPeriod && balance >= unlockThreshold24Hours) {
            if (twentyFourHourPeriod.order) {
                if (twentyFourHourPeriod.order.status === "NOT_WIN") {
                    console.log('24-hour trade lost. Retrying trading...');
                    await performTrading(token, 3); // Re-trade for 24-hour period
                } else {
                    console.log('24-hour trading is still in progress.');
                }
            } else {
                console.log('No active trade in 24-hour period. Starting new trade...');
                await performTrading(token, 3); // New trade for 24-hour period
            }
        }

    } catch (error) {
        console.error('Error processing unlocked trading periods: ', error.message);
    }
}

// Function to perform trading (game trading)
async function performTrading(token, periodId = 1) {
    const availableCoins = await fetchAvailableCoins(token);
    if (availableCoins.length > 0) {
        const { coinId, isShort } = selectBestCoin(availableCoins);
        console.log(`Selected coin for trading: ${coinId} (Short: ${isShort})`);

        // Perform a trade for the specified period
        await tradeCoin(token, coinId, isShort, periodId);
    } else {
        console.log('No available coins for trading.');
    }
}

// Function to fetch available coins
async function fetchAvailableCoins(token) {
    try {
        const response = await axios.get('https://api.miniapp.dropstab.com/api/order/coins', {
            headers: {
                'Authorization': `Bearer ${token}`,
                'Content-Type': 'application/json',
                'user-agent': 'Mozilla/5.0'
            }
        });

        return response.data;
    } catch (error) {
        console.error('Error fetching available coins: ', error.message);
        return [];
    }
}

// Function to select the best coin to trade based on potential
function selectBestCoin(coins) {
    let bestCoin = coins[0];
    for (const coin of coins) {
        if (coin.change24h > bestCoin.change24h) {
            bestCoin = coin;
        }
    }
    const isShort = bestCoin.change24h < 0; // Short if price dropped, otherwise long
    return { coinId: bestCoin.id, isShort };
}

// Function to perform the actual trade
async function tradeCoin(token, coinId, isShort, periodId) {
    const payload = {
        "coinId": coinId,
        "short": isShort,
        "periodId": periodId
    };

    try {
        const response = await axios.post('https://api.miniapp.dropstab.com/api/order', payload, {
            headers: {
                'Authorization': `Bearer ${token}`,
                'Content-Type': 'application/json',
                'user-agent': 'Mozilla/5.0'
            }
        });

        console.log(`Trade placed for coinId: ${coinId} (PeriodId: ${periodId}). Response: `, response.data);
    } catch (error) {
        console.error(`Error trading coin ${coinId} (PeriodId: ${periodId}): `, error.message);
    }
}

// Function to start the bot with header display
function startBot() {
    displayHeader();
    rl.question('Choose an option: 1. Get Token 🔑 2. Run Bot 🤖\n', async (answer) => {
        if (answer === '1') {
            console.log('🔄 Starting token retrieval for all accounts...');
            await getTokensForAllAccounts();
            console.log('✅ Token retrieval completed.');
            rl.close();
        } else if (answer === '2') {
            const tokens = readTokensFromFile(freshTokenFile);
            if (tokens.length > 0) {
                console.log('🤖 Starting bot with stored tokens...');
                await processAccounts();
                rl.close();
            } else {
                console.log('⚠️ Tokens not found. Please get tokens first.');
                rl.close();
            }
        } else {
            console.log('❌ Invalid option. Please choose 1 or 2.');
            rl.close();
        }
    });
}

// Start the bot
startBot();
