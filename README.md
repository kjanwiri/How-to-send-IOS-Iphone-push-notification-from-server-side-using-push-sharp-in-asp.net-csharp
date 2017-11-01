# How-to-send-IOS-Iphone-push-notification-from-server-side-using-push-sharp-in-asp.net-csharp

using Newtonsoft.Json.Linq;
using PushSharp.Apple;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;

namespace IOSPUshNotificationDemo12
{
    public partial class Demo : System.Web.UI.Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {

        }

        protected void btnSendNotification_Click(object sender, EventArgs e)
        {
            if(txtDeviceToken.Text!="" && txtMessage.Text!="")
            {
                SendPushNotification(txtDeviceToken.Text, txtMessage.Text);
            }
        }

        private void SendPushNotification(string deviceToken,string message)
        {
            try
            {

                //Get Certificate
                var appleCert = System.IO.File.ReadAllBytes(HttpContext.Current.Server.MapPath("~/Files/Certificate/IOS/Production_Certificate.p12"));

                // Configuration (NOTE: .pfx can also be used here)
                var config = new ApnsConfiguration(ApnsConfiguration.ApnsServerEnvironment.Production, appleCert, "1234567890");

                // Create a new broker
                var apnsBroker = new ApnsServiceBroker(config);

                // Wire up events
                apnsBroker.OnNotificationFailed += (notification, aggregateEx) =>
                {

                    aggregateEx.Handle(ex =>
                    {

                        // See what kind of exception it was to further diagnose
                        if (ex is ApnsNotificationException)
                        {
                            var notificationException = (ApnsNotificationException)ex;

                            // Deal with the failed notification
                            var apnsNotification = notificationException.Notification;
                            var statusCode = notificationException.ErrorStatusCode;
                            string desc = $"Apple Notification Failed: ID={apnsNotification.Identifier}, Code={statusCode}";
                            Console.WriteLine(desc);
                            lblStatus.Text = desc;
                        }
                        else
                        {
                            string desc = $"Apple Notification Failed for some unknown reason : {ex.InnerException}";
                            // Inner exception might hold more useful information like an ApnsConnectionException			
                            Console.WriteLine(desc);
                            lblStatus.Text = desc;
                        }

                        // Mark it as handled
                        return true;
                    });
                };

                apnsBroker.OnNotificationSucceeded += (notification) =>
                {
                    lblStatus.Text = "Apple Notification Sent successfully!";
                };

                var fbs = new FeedbackService(config);
                fbs.FeedbackReceived += (string devicToken, DateTime timestamp) =>
                {
                    // Remove the deviceToken from your database
                    // timestamp is the time the token was reported as expired
                };

                // Start Proccess 
                apnsBroker.Start();

                if (deviceToken != "")
                {
                    apnsBroker.QueueNotification(new ApnsNotification
                    {
                        DeviceToken = deviceToken,
                        Payload = JObject.Parse(("{\"aps\":{\"badge\":1,\"sound\":\"oven.caf\",\"alert\":\"" + (message + "\"}}")))
                    });
                }

                apnsBroker.Stop();

            }
            catch (Exception)
            {

                throw;
            }
        }
    }
}
